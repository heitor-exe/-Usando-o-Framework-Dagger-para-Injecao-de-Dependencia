Usando o Framework Dagger para Injeção de Dependência
========
Roteiro Prático de Laboratório de Engenharia de Software 1
-------
Alunos: Antônio José Peixoto Chaves --- Matrícula:20193002329
-------

Introdução
-------
Em engenharia de software, a injeção de dependências é uma técnica onde um objeto (ou método estático) fornece as dependências de outro objeto. Uma dependência é um objeto que pode ser usado (um serviço).

Quando a classe A usa alguma funcionalidade da classe B, diz-se que a classe A tem uma dependência da classe B.

Em Java, antes de poder usar métodos de outras classes, primeiro precisamos criar o objeto daquela classe (ou seja, a classe A precisa criar uma instância da classe B).

Desse modo, transferir a tarefa de criação do objeto a outra entidade e usar diretamente a dependência é chamado de injeção de dependência.

Basicamente, existem três tipos de injeção de dependências:
1- injeção do construtor: as dependências são fornecidas por meio de um construtor da classe.
2- injeção pelo setter: o cliente expõe o método setter que o injetor usa para injetar a dependência.
3- injeção de interface: a dependência fornece um método injetor, que injetará uma dependência em qualquer cliente que for passado a ele. Os clientes devem implementar uma interface que expõe um método setter que aceita a dependência.
Desse modo, agora é responsabilidade da injeção de dependência:

- Criar os objetos
- Saber quais classes necessitam desses objetos
- Fornecer todos esses objetos

Bibliotecas e frameworks que implementam a injeção de dependência:
- Spring (Java)
- Google Guice (Java)
- Dagger (Java and Android)
- Castle Windsor (.NET)
- Unity(.NET)

Dagger
========
Princípios básicos do Dagger
-------
A injeção de dependência manual ou os localizadores de serviço em um app Android podem ser problemáticos, dependendo do tamanho do projeto. Você pode limitar a complexidade do seu projeto conforme ele se expande usando o Dagger para gerenciar dependências.

O Dagger gera automaticamente um código que imita o que você teria programado manualmente. Como o código é gerado no momento da compilação, ele é rastreável e tem melhor desempenho que outras soluções baseadas em reflexões.

As vantagens de usar o Dagger
O Dagger libera você do processo tedioso de ter que escrever códigos de texto clichê propenso a erros:
- Gerar código AppContainer (gráfico do aplicativo) que você implementou manualmente na seção da ID manual.
- Criar fábricas para as classes disponíveis no gráfico do aplicativo. É assim que as dependências são satisfeitas internamente.
- Escolher reutilizar uma dependência ou criar uma nova instância usando escopos.
- Criar contêineres para fluxos específicos, como você fez com o fluxo de login na seção anterior usando subcomponentes do Dagger. Isso melhora o desempenho do app ao liberar objetos na memória quando eles não são mais necessários.

O Dagger faz tudo isso automaticamente no tempo de compilação, contanto que você declare as dependências de uma classe e especifique como elas vão ser satisfeitas usando anotações. O Dagger gera um código semelhante ao que você escreveria manualmente. Internamente, o Dagger cria um gráfico de objetos que ele pode referenciar para encontrar a maneira de fornecer uma instância de uma classe. Para cada classe no gráfico, o Dagger gera uma classe do tipo fábrica class que a ferramenta usa internamente para receber instâncias desse tipo.

No momento da compilação, o Dagger revisa seu código e cria e valida gráficos de dependência, garantindo que:
- as dependências de cada objeto possam ser atendidas, de modo que não haja exceções de tempo de execução.
- não haja ciclos de dependência, portanto, não haja loops infinitos.
- Gera as classes usadas no tempo de execução para criar os objetos reais e suas respectivas dependências.

Dagger 2 Api
-------
A versão 2 do Dagger expõe algumas anotações especiais que são:
@Module, classe que irá prover as dependências
@Provides, para os métodos provedores da classe de módulos
@Inject, para requisitar um dependência (use no construtor, atributos públicos ou métodos set)
@Component, funciona como um ponte entre os módulos(@Module) e a injeção (@Inject).

Implementando o Dagger
-------
Passo 1: Identificar objetos e suas dependências
-------
Para este tutorial teremos duas classes, Usuário que represente um usuário do nosso aplicativo e Perfil, que representa o seu perfil de usuário.

```java
class Usuario {
		
	public Perfil perfil;	

	public Usuario(Perfil perfil){
	     this.perfil = perfil;
	}

}

class Perfil{
  String nome;

  public String getNome(){
       return this.nome;
  }
}

class UsuarioFree extends Perfil{
     public UsuarioFree(){
         this.nome ="Usuário Free";
     }
}

class UsuarioPremium extends Perfil{
     public UsuarioFree(){
         this.nome ="Usuário Premium";
     }
}
```
Passo 2 : Criar nossa classe de Módulo
-------
Classes anotadas com @Module, devem conter métodos anotados com @Provides, esses métodos é que serão chamados na hora que as dependências forem injetadas.

```java
import javax.inject.Singleton;
 
import dagger.Module;
import dagger.Provides;
 
/**
 * Created by kerry on 14/02/15.
 */
 
@Module
public class UsuarioModule {
 
    @Provides @Singleton
    Perfil providePerfil(){
        return new PerfilPremium();
    }
 
    @Provides @Singleton
    Usuario provideUsuario(){
        return new Usuario(new PerfilPremium());
    }
}
```

Aqui criamos dois métodos, o primeiro provê um Objeto Perfil, que é independente, e o outro provê um objeto Usuário que possui como dependência um Perfil.

A anotação @Singleton indica que em toda aplicação existirá apenas um instância do objeto. Isto é sempre que precisarmos de um objeto usuário, será retorna a mesma instância.


Passo 3 : Requisitando a injeção de dependências dentro do Objeto Dependente
-------
Agora nossa classe módulo possui métodos provedores para as nossas diferentes classes. Nossa classe usuário necessita de um Perfil, então precisamos indicar isso usando a anotação @Inject no nosso construtor (poderíamos criar um método setPerfil, ou deixar o atributo perfil como público e anotá-lo).

```java
@Inject
public Usuario(Perfil perfil){
    this.perfil = perfil;
}
```

Passo 4: Conecctar os Módulos com os Injetores
-------
A conexão entre os Módulos (@Module) provedores e as classes que estão requisitando objetos (@Inject) se dá através dos componentes (@Component), que deve ser uma interface ou classe abstrata.

```java
import javax.inject.Singleton;
 
import dagger.Component;
 
@Singleton
@Component(modules = {UsuarioModule.class})
public interface UsuarioComponent {
 
    Usuario provideUsuario();
 
}
```

Ao fazer a anotação @Component, você deve especificar quais módulos estarão disponíveis naquele componente(Separar módulos com ‘,’).


Com esta interface, o Dagger irá implementar os métodos abstratos e adicionar alguns métodos mais, que serão úteis ao nosso desenvolvimento.


Passo 5: Utilizar a interface anotada com @Component para obter os objetos
-------
Agora que está tudo configurado e conectado podemos obter uma instancia dessa interface e invocar seus métodos para obter nossos objetos, através dos provedores.

Para isso irei implementar as chamadas no método OnCreate da nossa MainActivity.
```java
package net.ramonsilva.tutorial.dagger;
 
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.widget.Toast;
 
import net.ramonsilva.tutorial.component.DaggerUsuarioComponent;
import net.ramonsilva.tutorial.component.UsuarioComponent;
import net.ramonsilva.tutorial.model.Usuario;
import net.ramonsilva.tutorial.module.UsuarioModule;
 
public class MainActivity extends ActionBarActivity {
 
    Usuario usuario;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        UsuarioComponent component = DaggerUsuarioComponent.builder().usuarioModule(new UsuarioModule()).build();
 
        usuario = component.provideUsuario();
 
        Toast.makeText(this, String.valueOf(usuario.getNome()), Toast.LENGTH_SHORT).show();
    }
}
```

Quando queremo criar uma instancia da interface anotada com @Component, você deve chamar o método build do classe Gerada pelo Dagger, essa classe terá o nome Dagger<Nome_do_Component>.


A partir desta instancia podemos chamar os métodos provedores de objetos, e receberemos nossas instancias de objetos já com todas as dependências injetadas.
```java
usuario = component.provideUsuario();
```

Conclusão
=======
A injeção de dependências é um padrão que deve-se usar desde de cedo no seu projeto, porém se for necessário aplicá-lo mais tarde no projeto, será necessário algumas refatorações, porém todo esse trabalho será recompensado com um código mais limpo e fácil de manter e testar.






License
-------

    Copyright 2012 Square, Inc.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.



 [1]: http://square.github.com/dagger/
 [dl-dagger]: http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.squareup.dagger%22%20a%3A%22dagger%22
 [dl-javapoet]: http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.squareup%22%20a%3A%22javapoet%22
 [dl-inject]: http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22javax.inject%22%20a%3A%22javax.inject%22
 [snap]: https://oss.sonatype.org/content/repositories/snapshots/
