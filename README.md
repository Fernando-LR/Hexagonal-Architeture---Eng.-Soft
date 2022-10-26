# Hexagonal Architeture

## Um resumo sobre Arquitetura Hexagonal
O conceito de Arquitetura Hexagonal foi proposto por Alistair Cockburn, em meados dos anos 90, em um artigo postado na primeira wiki que foi desenvolvida, chamada WikiWikiWeb (cujos artigos tratavam principalmente de temas relacionados com Engenharia de Software).

Os objetivos de uma Arquitetura Hexagonal são parecidos com os de uma Arquitetura Limpa, onde a ideia é construir sistemas que favorecem a reusabilidade de código, alta coesão, baixo acoplamento, independência de tecnologia e que são mais fáceis de serem testados.

Uma Arquitetura Hexagonal divide as classes de um sistema em dois grupos principais:
* Classes de domínio, isto é, diretamente relacionadas com o negócio do sistema.
* Classes relacionadas com infraestrutura, tecnologias e responsáveis pela integração com sistemas externos (tais como bancos de dados).

Além disso, em uma Arquitetura Hexagonal, classes de domínio não devem depender de classes relacionadas com infraestrutura, tecnologias ou sistemas externos. A vantagem dessa divisão é desacoplar esses dois tipos de classes.

Assim, as classes de domínio não conhecem as tecnologias – bancos de dados, interfaces com usuário e quaisquer outras bibliotecas – usadas pelo sistema. Consequentemente, mudanças de tecnologia podem ser feitas sem impactar as classes de domínio. Talvez ainda mais importante, as classes de domínio podem ser compartilhadas por mais de uma tecnologia. Por exemplo, um sistema pode ter diversas interfaces (Web, mobile, etc).

Em uma arquitetura hexagonal, a comunicação entre as classes dos dois grupos é mediada por adaptadores, isto é, por classes que implementam o padrão de projeto de mesmo nome.

Visualmente, a arquitetura é representada por meio de dois hexágonos concêntricos. No hexágono interno, ficam as classes do domínio. No hexágono externo, ficam os adaptadores. Por fim, as classes de interface com o usuário, classes de tecnologia ou de sistemas externos ficam fora desses dois hexágonos.

![Arquitetura Hexagonal](https://engsoftmoderna.info/artigos/figs/arquitetura-hexagonal.svg)
## Adaptadores e Portas
Em uma Arquitetura Hexagonal, o termo porta designa as interfaces usadas para comunicação com as classes de domínio (veja que interface aqui significa interface de programação; por exemplo, uma interface de Java).

Existem dois tipos de portas:

* **Portas de entrada:** são interfaces usadas para comunicação de fora para dentro, isto é, quando uma classe externa precisa chamar um método de uma classe de domínio. Logo, essas portas declaram os serviços providos pelo sistema, isto é, serviços que o sistema oferece para o mundo exterior.

* **Portas de saída:** são interfaces usadas para comunicação de dentro para fora, isto é, quando uma classe de domínio precisa chamar um método de uma classe externa. Logo, essas portas declaram os serviços requeridos pelo sistema, isto é, serviços do mundo exterior que são necessários para o funcionamento do sistema.

O importante é que as portas são independentes de tecnologia. Portanto, elas estão localizadas no hexágono interior.

Por outro lado, os sistemas externos, normalmente, usam alguma tecnologia, seja ela de comunicação (REST, gRPC, GraphQL, etc), de bancos de dados (SQL, noSQL, etc), de interação com o usuário (Web, mobile, etc), etc.

Daí a necessidade de componentes localizados no hexágono mais externo da arquitetura – os adaptadores –, os quais atuam de um dos dois modos a seguir:
* Eles recebem chamadas de métodos vindas de fora do sistema e encaminham essas chamadas para métodos adequados das portas de entrada.
* Eles recebem chamadas vindas de dentro do sistema, isto é, das classes de domínio, e as direcionam para um sistema externo, tais como um banco de dados, um outro sistema da organização ou mesmo de terceiros.


## Praticando

Para entender melhor como é uma arquitetura hexagonal, vamos considerar um exemplo de uma aplicação para uma pizzaria. Nessa aplicação, devemos ter as seguintes características:
* Lista de todas as pizzas disponíveis
* Inserir um nova sabor de pizza ao menu
* Obter uma determinada pizza através de seu nome

O objeto de domínio é a parte central da arquitetura hexagonal. Pode ter estado e comportamento. Este objeto não depende de nenhum dos componentes do aplicativo. Qualquer alteração no objeto de domínio ocorrerá se houver uma alteração no próprio requisito de negócios.

Para isso, vamos criar a classe Pizza:

```
public class Pizza {
    private String nome;
    private int price;
    private String[] recheio;
    // Getters e Setters
    // Construtor
}
```
## Parte 2: Portas
As portas na arquitetura hexagonal referem-se às interfaces que permitem o fluxo de entrada ou saída, sendo assim elas são divididas em dois tipos, portas de entrada e portas de saída. 

* **2.1: Porta de Entrada**
Uma porta de entrada expõe a funcionalidade do aplicativo em contato com o mundo externo, como o uso de um API.
Para esse exemplo, vamos definir a interface PizzaService.
```
public interface PizzaService {
      public void createPizza(Pizza pizza);
      public Pizza getPizza(String name);
      public List<Pizza> loadPizza();
}
```

* **2.2: Porta de Saída**
As portas de saída, por sua vez, são usadas para conectar com repositórios externos, como o acesso ao banco de dados. Para esse exemplo, vamos definir o PizzaDAO.
```
public interface PizzaRepo {
      public void createPizza(Pizza pizza);
      public Pizza getPizza(String name);
      public List<Pizza> getAllPizza();
}
```

## Parte 3: Adaptadores
Os adaptadores são a parte externa do aplicativo, como GUI, API, DAO e Web, eles se referem às classes de implementação de suas respectivas portas na arquitetura hexagonal, tendo sua interação com o aplicativo através das portas aprendidas na Parte 2.

Os adaptadores facilitam a troca de uma camada do aplicativo, sendo necessário apenas adicionar um adaptador com uma porta de entrada ou saída(Parte 2.1 e 2.2).

* **3.1: Adaptadores Primários:**
Ou adaptadores de entrada, conduzem o aplicativo executando a sua parte principal utilizando as portas de entrada.

Para esse exemplo, vamos definir a classe PizzaRestContoller como um controlador REST como nosso adaptador primário. Ele fornece endpoints para criar e buscar pizzas e também implementa PizzaRestUI (Webview). Além disso, usa PizzaService (porta de entrada) para invocar diferentes métodos.
```
@RestController
@RequestMapping(value="/pizza")
public class PizzaRestController implements PizzaRestUI {
      @Autowired
      private PizzaService pizzaService;
      @Override
      public void createPizza(@RequestBody Pizza pizza) {
            pizzaService.createPizza(pizza);
      }
      @Override
      public Pizza getPizza(@PathVariable String name) {
            return pizzaService.getPizza(name);
      }
      @Override
      public List<Pizza> listPizza() {
            return pizzaService.loadPizza();
      }
}
```
* **3.2 Adaptadores Secundários:** 
Ou adaptadores de saída, implementam a interface Esses adaptadores fornecem uma implementação para acessar os componentes secundários de um aplicativo, como bancos de dados, filas de mensagens, etc. Enquanto a camada de serviço implementa a porta de entrada, uma porta de saída é implementada usando a camada de persistência.

No nosso caso, PizzaRepoImpl é o adaptador de saída que implementa PizzaDAO (porta de saída).
```
@Repository
public class PizzaRepoImpl implementa PizzaDAO {
      private Map<String, Pizza> pizzaStore = new HashMap<String, Pizza>();
      @Override
      public void createPizza(Pizza pizza) {
            pizzaStore.put(pizza.getName(), pizza);
      }
      @Override
      public Pizza getPizza(String name) {
            return pizzaStore.get(name);
      }
      @Override
      public List<Pizza> getAllPizza() {
            return pizzaStore.values().stream().collect(Collectors.toList());
      }
}
```
Após os adaptadores e portas serem criados, é possível fazer o teste através usando a API.

Para isso iremos testar o POST através do link: 
http://localhost:8080/pizza-service/pizza/
```
{
   "name" : "Margherita",
   "price": "25",
   "toppings" : ["tomate","cebola","pepino","jalapeno"]
}
```

Além, disso podemos testar o GET através do link:
http://localhost:8080/pizza-service/pizza/Margherita
```
{
   "name": "Margherita",
   "price": 25,
   "toppings": [
       "tomate",
       "cebola",
       "pepino",
       "jalapeno"
   ]
}
```
## 4: Conclusão
Podemos concluir, que a arquitetura hexagonal é utilizada para simplificar o design do aplicativo tendo partes separadas para controlar os componentes externos e internos, possuindo um alto grau de desacoplamento. Além do fato de, por ser baseada em portas, permite a fácil adaptação de novas utilidades e protocolos do aplicativo no futuro.
