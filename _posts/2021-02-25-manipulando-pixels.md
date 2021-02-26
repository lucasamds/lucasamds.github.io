---
layout: post
title: Manipulando pixels em uma imagem com Python
---


<div class="message">
  Olá, este é o primeiro post de uma série voltada para o processamento digital de imagens utilizando Python e OpenCV.
</div>

Neste post vamos realizar algumas manipulações com os pixels de uma dada imagem. Para tal, iremos utilizar a linguagem de programação *Python*, juntamente com as bibliotecas *OpenCV* e *Numpy*. 

## Encontrando o negativo

Nosso primeiro objetivo é solicitar dois pontos **P<sub>1</sub>** e **P<sub>2</sub>** que estejam localizados dentro dos limites da imagem passada, em seguida devemos exibir a imagem de tal forma que os pixels contidos na região formada pelo retângulo de vértices opostos, definidos pelos pontos **P<sub>1</sub>** e **P<sub>2</sub>**, serão exibidos com o negativo da imagem na região correspondente. Para esse estudo iremos utilizar a imagem <a href="https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/cat.jpg" target="_blank">*cat.jpg*</a>.

##### Listagem 1. negativo.py
{% highlight python %}
import cv2 as cv
import sys
import numpy as np
#Leitura da imagem
img = cv.imread("imagens/cat.jpg", cv.IMREAD_GRAYSCALE)

#Caso a imagem não seja encontrada
if img is None:
    sys.exit("O arquivo não foi encontrado.")

width = len(img[0])
height = len(img)

#Recebe os dois pontos do usuário
x1,y1,x2,y2 = input(f"Diga as coordenadas x e y de dois pontos na imagem ({width}x{height}): ").split()

x1 = int(x1)
x2 = int(x2)
y1 = int(y1)
y2 = int(y2)

if x1 > x2:
    aux = x1
    x1 = x2
    x2 = aux
if y1 > y2:
    aux = y1
    y1 = y2
    y2 = aux

#Modificando pixels da imagem
for i in range(x1,x2):
    for j in range(y1,y2):
        img[i,j] = 255 - img[i,j]

#Cria uma janela que se ajusta ao tamanho da imagem
cv.namedWindow("Imagem", cv.WINDOW_AUTOSIZE)
#Aprensentando a imagem
cv.imshow("Imagem", img)
k = cv.waitKey(0)

{% endhighlight %}

## Descrição do programa negativo.py

{% highlight python %}
import cv2 as cv
import sys
import numpy as np

{% endhighlight %}

Como já foi dito anteriormente, para resolver este problema iremos utilizar as bibliotecas OpenCV e Numpy. Devido a alta otimização da biblioteca Numpy para operações numéricas, torna-se quase que indispensável o seu uso para as operações que iremos realizar com o OpenCV, devido a isso as novas versões da biblioteca possibilitam que qualquer estrutura array do OpenCV possa ser convertido em um array do Numpy, o contrário também é verdadeiro. 

{% highlight python %}
#Leitura da imagem
img = cv.imread("imagens/cat.jpg", cv.IMREAD_GRAYSCALE)

#Caso a imagem não seja encontrada
if img is None:
    sys.exit("O arquivo não foi encontrado.")

width = len(img[0])
height = len(img)
{% endhighlight %}

Em seguida iremos fazer a leitura da imagem, a função `imread()` é chamada recebendo dois parâmetros: o primeiro indica o caminho da imagem que será aberta; o segundo informa como a imagem deve ser interpretada, neste caso iremos trabalhar com a imagem em tons de cinza, logo a flag `cv.IMREAD_GRAYSCALE` é fornecida. Após a leitura fazemos uma verificação, caso tenha ocorrido algum problema durante a leitura da imagem, o programa mostra uma mensagem de erro e é encerrado; caso tudo tenha dado certo, vamos consultar o tamanho da imagem para uso futuro, aqui usamos a função `len()`.

{% highlight python %}
x1,y1,x2,y2 = input(f"Diga as coordenadas x e y de dois pontos na imagem ({width}x{height}): ").split()

x1 = int(x1)
x2 = int(x2)
y1 = int(y1)
y2 = int(y2)
{% endhighlight %}

O próximo passo é obter as coordenadas dos pontos que iremos utilizar para identificar os pixels que serão modificados, note que mostramos na tela as dimensões da imagem lida, assim fica mais fácil de escolher os valores válidos para as coordenadas. O comando `input()` vai nos retornar uma string que possui os dados que foram digitados, como queremos armazenar os valores de forma segmentada, utilizamos a função `split()` para realizar "quebras" na string, como nenhum parâmetro foi passado, a função por padrão irá realizar a quebra da string sempre que encontrar um espaço. Em seguida, realizamos a conversão das entradas para o tipo inteiro através de *Cast*.

Antes de escolher as coordenadas dos pontos, é importante ter em mente em como o OpenCV acessa os elementos da imagem, veja a figura abaixo.

|![eixos.png](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/eixos.png)|
|:-------:|
| *Figura 1, Sistema referencial adotado pelo OpenCV* |



> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.


Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/poole/poole/issues/new">Open an issue.</a>
