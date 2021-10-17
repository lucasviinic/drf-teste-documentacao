# API com Django 3: Aprofundando em testes e documentação

## Aula 01: Tipos de Testes

É possível carregar dados iniciais para os modelos através das fixtures. Para isso, dentro de cada APP, podemos criar uma pasta chamada fixtures com dados no formato JSON, 
por exemplo, e carregá-los executando o comando manage.py loaddata seguido do nome do arquivo que contém os dados.

1. **Teste Manual:** Uma pessoa está executando.
2. **Teste automatizado:**

    - **Teste Unitário:** Testa métodos ou funções individuais. Concentram-se nas preocupações da pessoa que desenvolve e geralmente são pequenos. 
São executados em microssegundos e testam uma única parte do projeto.
    - **Teste Funcional:** Testar os requisitos do negócio. O teste funcional é focado na experiência do usuário, pode ser grande e levar mais tempo
quando comparado aos testes de unidade. Geralmente, simula aquilo que um usuário faria.
    - **Teste de Integração:** Testar diferentes módulos ou serviços utilizados por um aplicativo.
    
## Aula 02: Teste Unitário

1. **Testando Modelo:**

  Na pasta do app, no caso do curso, a pasta "aluraflix", criamos a pasta de testes nomeada "tests" no plural, em seguida criamos o `__init__.py`, isso
pois iremos ter vários códigos a serem executados, então, criamos o primeiro código de teste, par testar os modelos, portanto `test_models.py`. <p> Em um teste
de unidade em vez de `self.programa = Programa()` fazemos `self.programa = Programa.objects.create()`, isso pois, estamos definindo um teste de unidade
e não de integração, sendo assim precisamos apenas de uma instância do objeto de teste. </p>

2. **Testando Serializer:**

  De forma semelhante, é criado o código de teste `test_serializers.py`. Em seguida, na construção do caso de teste, são passados todos os campos do programa
com seus respectivos valores. <p> Definimos `self.serializers = ProgramaSerializer(instance=self.programa)`, neste trecho definimos qual a classe serializer
e a instância que estamos serializando, portanto, atribuímos a instância `self.programa` ao parâmetro `instance`. A configuração do caso de teste fica assim: </p>

```
def setUp(self):
    self.programa = Programa(
        titulo = 'Procurando ninguém em latim',
        data_lancamento = '2003-07-04',
        tipo = 'F',
        likes = 2340,
        dislikes = 40
    )
    self.serializer = ProgramaSerializer(instance=self.programa)
```

Para testar os campos serializados, armazenamos os dados do serializer e verificamos  se os campos do serializer são os mesmos passados no teste, por meio da expressão. 

```
def test_verifica_os_serializados(self):
    '''Teste que verifica os campos que estão sendo serializados'''
    data = self.serializer.data
    self.assertEqual(set(data.keys()), set(['titulo', 'tipo', 'data_lancamento', 'likes']))
```

Aqui vemos o uso do `set()`, muito importante para que  qualquer adição ou remoção de campos do serializer seja percebida pelo teste. Para testar se o conteúdo é o mesmo
esperado no teste, é simples, recebemos os dados do serializer e comparamos os valores de forma bem intuitiva, comparamos os valores recebidos com os valores do programa
definido no caso de teste.

```
def test_verifica_conteudo_dos_campos_serializados(self):
    '''Teste que verifica o conteudo dos campos serializados'''
    data = self.serializer.data
    self.assertEqual(data['titulo'], self.programa.titulo)
    self.assertEqual(data['data_lancamento'], self.programa.data_lancamento)
    self.assertEqual(data['tipo'], self.programa.tipo)
    self.assertEqual(data['likes'], self.programa.likes)
```

## Aula 03: Teste de Integração

<p> A estudar </p>

## Aula 04: Testando a API

<p> A estudar </p>

## Aula 05: Documentando a API com Swagger

<p> A estudar </p>


_**Nota:** Anotações referentes ao curso de aprofundamento em testes da formação Django REST da Alura_

