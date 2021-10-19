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

**Código completo**

```
from django.test import TestCase
from aluraflix.models import Programa


class ProgramaModelTestCase(TestCase):
    
    def setUp(self):
        self.programa = Programa(
            titulo = 'Procurando ninguém em latim',
            data_lancamento = '2003-07-04'
        )

    def test_verifica_atributos_do_programa(self):
        '''Teste que verifica os atributos de um programa com valores default'''
        self.assertAlmostEqual(self.programa.titulo, 'Procurando ninguém em latim')
        self.assertAlmostEqual(self.programa.tipo, 'F')
        self.assertAlmostEqual(self.programa.data_lancamento, '2003-07-04')
        self.assertAlmostEqual(self.programa.likes, 0)
        self.assertAlmostEqual(self.programa.dislikes, 0)

    
```

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

**Código completo**

```
from django.test import TestCase
from aluraflix.models import Programa
from aluraflix.serializers import ProgramaSerializer


class ProgramaSerializerTestCase(TestCase):

    def setUp(self):
        self.programa = Programa(
            titulo = 'Procurando ninguém em latim',
            data_lancamento = '2003-07-04',
            tipo = 'F',
            likes = 2340,
            dislikes = 40
        )
        self.serializer = ProgramaSerializer(instance=self.programa)

    def test_verifica_os_serializados(self):
        '''Teste que verifica os campos que estão sendo serializados'''
        data = self.serializer.data
        self.assertEqual(set(data.keys()), set(['titulo', 'tipo', 'data_lancamento', 'likes']))

    def test_verifica_conteudo_dos_campos_serializados(self):
        '''Teste que verifica o conteudo dos campos serializados'''
        data = self.serializer.data
        self.assertEqual(data['titulo'], self.programa.titulo)
        self.assertEqual(data['data_lancamento'], self.programa.data_lancamento)
        self.assertEqual(data['tipo'], self.programa.tipo)
        self.assertEqual(data['likes'], self.programa.likes)
```

## Aula 03: Teste de Integração

Dando início ao teste de integração, testamos a autenticação de um usuário com as credenciais corretas, para isso precisamos importar o objeto user, fazendo `from django.contrib.auth.models import User` e o método authenticate para realizar a verificação `from django.contrib.auth import authenticate`.

```
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from django.contrib.auth import authenticate


class AuthenticationUserTestCase(APITestCase):

    def setUp(self):
        self.user = User.objects.create_user('c3po', password='123456')

    def test_autenticacao_user_com_credenciais_corretas(self):
        '''Teste que verifica a autenticação de um user com as credenciais corretas'''
        user = authenticate(username='c3po', password='123456')
        self.assertTrue((user is not None) and user.is_authenticated)
```

</p> Certo, agora iremos testar a requisição GET para um usuário não logado, e para isso é necessário passar a URL que vai realizar a requisição, então, `from django.urls import reverse`, esse módulo é necessário pois irá expandir todas as possíveis URLs. </p>

No setup do caso de teste, adicionamos a instância `self.list_url = reverse('programas-list')`, ou seja, a lista com todas as possíveis requisições do recurso programas. 
Em seguida, realizamos o seguinte teste para verificar o caso de uma requisição GET com o usuário não logado:
    
```
def test_requisicao_get_nao_autorizada(self):
    '''Teste que verifica uma requisição GET sem autenticar'''
    response = self.client.get(self.list_url)
    self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
```

<p> Neste teste, realizamos uma requisição GET para o recurso programas e verificamos se o status code é 401, uma requisição não autorizada. Em seguida, sem grandes mistérios verificamos a autenticação nos casos de username ou senha incorretos </p>

```
def test_autenticacao_de_user_com_username_incorreto(self):
    '''Teste que verifica autenticação de um user com username incorreto'''
    user = authenticate(username='c3pp', password='123456')
    self.assertFalse((user is not None) and user.is_authenticated)

def test_autenticacao_de_user_com_password_incorreto(self):
    '''Teste que verifica autenticação de um user com password incorreto'''
    user = authenticate(username='c3po', password='123455')
    self.assertFalse((user is not None) and user.is_authenticated)
```

Em seguida testamos uma requisição em que é forçada a autenticação do usuário

```
def test_requisicao_get_com_user_autenticado(self):
    '''Teste que verifica uma requisição GET de um user autenticado'''
    self.client.force_authenticate(self.user) #Forçamos que o cliente da requisição seja autenticado
    response = self.client.get(self.list_url)
    self.assertEqual(response.status_code, status.HTTP_200_OK)
```

**Código completo**

```
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from django.contrib.auth import authenticate
from django.urls import reverse
from rest_framework import status


class AuthenticationUserTestCase(APITestCase):

    def setUp(self):
        self.list_url = reverse('programas-list')
        self.user = User.objects.create_user('c3po', password='123456')

    def test_autenticacao_user_com_credenciais_corretas(self):
        '''Teste que verifica a autenticação de um user com as credenciais corretas'''
        user = authenticate(username='c3po', password='123456')
        self.assertTrue((user is not None) and user.is_authenticated)

    def test_requisicao_get_nao_autorizada(self):
        '''Teste que verifica uma requisição GET não autorizada'''
        response = self.client.get(self.list_url)
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

    def test_autenticacao_de_user_com_username_incorreto(self):
        '''Teste que verifica autenticação de um user com username incorreto'''
        user = authenticate(username='c3pp', password='123456')
        self.assertFalse((user is not None) and user.is_authenticated)

    def test_autenticacao_de_user_com_password_incorreto(self):
        '''Teste que verifica autenticação de um user com password incorreto'''
        user = authenticate(username='c3po', password='123455')
        self.assertFalse((user is not None) and user.is_authenticated)

    def test_requisicao_get_com_user_autenticado(self):
        '''Teste que verifica uma requisição GET de um user autenticado'''
        self.client.force_authenticate(self.user) #Forçamos que o cliente da requisição seja autenticado
        response = self.client.get(self.list_url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
```

## Aula 04: Testando a API

### Fixtures nos Testes

Sempre que realizávamos um teste era criado um banco de dados padrão que, ao final dos testes, era destruído. Porém, é possível pegar as fixtures e utilizálas nos nossos
testes. Para isso, declaramos `fixtures = ['programas_iniciais']`. Em seguida testamos se realmente os dados da fixture estão no banco de dados de teste.

**Código completo**
```
from django.test import TestCase
from aluraflix.models import Programa


class FixturesDataTestCase(TestCase):

    fixtures = ['programas_iniciais']

    def test_verifica_carregamento_da_fixture(self):
        programa_bizarro = Programa.objects.get(pk=1)
        todos_os_programas = Programa.objects.all()
        
        self.assertEqual(programa_bizarro.titulo, 'Coisas bizarras')
        self.assertEqual(len(todos_os_programas), 9)
```

### Testando API no Postman

Em seguida, testamos a API no Postman, verificando se o statuscode da requisição era o esperado. Além disso, verificamos também se o tipo de dado da resposta estava no formato desejado.

**Screenshot**

<img src="https://i.imgur.com/tRqLxVc.png" width="100%" />

## Aula 05: Documentando a API com Swagger

<p> A estudar </p>


_**Nota:** Anotações referentes ao curso de aprofundamento em testes da formação <a href="https://cursos.alura.com.br/formacao-django-rest">Django REST APIs</a> da Alura_

