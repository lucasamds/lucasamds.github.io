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

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/eixos.png)
*Figura 1. Sistema referencial adotado pelo OpenCV*

{% highlight python %}
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

{% endhighlight %}

O trecho de código acima prepara os pontos em que iremos operar, os dois testes de seleção realizam trocas de valores entre as coordenadas que foram lidas, garantindo assim que partiremos do canto superior esquerdo da nossa região de interesse, indo até o canto inferior direito. Já o conjunto de laços de repetição realiza a troca de valores dos pixels da imagem, como estamos interessados em gerar o negativo da imagem, temos que o novo valor do pixel de interesse será a diferença entre o maior tom de cinza suportado (255) e o seu valor original.

{% highlight python %}

#Cria uma janela que se ajusta ao tamanho da imagem
cv.namedWindow("Imagem", cv.WINDOW_AUTOSIZE)
#Aprensentando a imagem
cv.imshow("Imagem", img)
k = cv.waitKey(0)

{% endhighlight %}

Agora que as operações na imagem já foram realizadas, nos resta apresentar os resultados obtidos. Primeiramente chamamos a função `namedWindow()`, passando um nome para a janela que será criada e a flag `WINDOW_AUTOSIZE`, esta flag indica que a janela irá se adequar ao tamanho da imagem. Em seguida a função `imshow()` irá apresentar o conteúdo armazenado em `img` na janela criada. A função `waitKey(delay)` espera indefinidamente pela ativação de uma tecla quando **delay &le; 0**, ou espera **delay** segundos caso contrário, neste caso a imagem fica aguardando que alguma tecla seja pressionada. 

Ao rodar o código a seguinte mensagem é apresentada:

```Diga as coordenadas x e y de dois pontos na imagem (640x427):```




Como discutimos anteriormente, o programa nos diz as dimensões da imagem lida, caso os valores de entrada sejam `0 0 427 320`, por exemplo, temos o seguinte resultado:

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/negativo.png)
*Figura 2. Imagem cat.jpg em tons de cinza antes e depois de aplicar o negativo*


## Trocando regiões

Agora vamos tentar realizar um segundo procedimento, nosso objetivo é trocar os pixels de região, de modo que os valores que estavam armazenados no quadrante superior esquerdo troquem de posição com os pixels do quadrante inferior direito, de forma semelhante, faremos a troca de valores entre os quadrantes superior direito e inferior esquerdo. Neste segundo experimento não será necessário fazer a leitura de valores de entrada adicionais, apenas da própria imagem que desejamos realizar o processamento.

##### Listagem 2. trocaregioes.py
{% highlight python %}
import cv2 as cv
import sys
import numpy as np

#Leitura da imagem
img = cv.imread("imagens/cat.jpg", cv.IMREAD_GRAYSCALE)

#Caso a imagem não seja encontrada
if img is None:
    sys.exit("O arquivo não foi encontrado.")

#Cria uma janela que se ajusta ao tamanho da imagem
cv.namedWindow("Imagem", cv.WINDOW_AUTOSIZE)

#Cria cópia da imagem
img2 = np.zeros((len(img),len(img[0])),dtype=np.uint8)

#Econtrando o meio da imagem
meioh = int(len(img[0])/2)
meiov = int(len(img)/2)


#Modificando quadrantes
for i in range(0,meiov):
    for j in range(meioh,len(img[0])):
        img2[i,j] = img[meiov + i,j - meioh]

for i in range(meiov,len(img)):
    for j in range(meioh,len(img[0])):
        img2[i,j] = img[i - meiov,j - meioh]

for i in range(0,meiov):
    for j in range(0,meioh):
        img2[i,j] = img[meiov + i,meioh + j]

for i in range(meiov,len(img)):
    for j in range(0,meioh):
        img2[i,j] = img[i-meiov,meioh + j]

#Aprensentando a imagem
cv.imshow("Imagem", img2)
cv.waitKey(0)
{% endhighlight %}

## Descrição do programa trocaregioes.py

A parte inicial do código segue a mesma ideia do caso anterior, continuamos trabalhando com a imagem em tons de cinza, a primeira diferença que encontramos entre os dois programas aparece logo após a verificação de leitura da imagem.

{% highlight python %}

#Cria cópia da imagem
img2 = np.zeros((len(img),len(img[0])),dtype=np.uint8)

#Econtrando o meio da imagem
meioh = int(len(img[0])/2)
meiov = int(len(img)/2)

{% endhighlight %}

Aqui criamos um array utilizando a função `zeros()` da biblioteca Numpy, a ideia aqui é criar uma nova imagem que possua as regiôes trocadas, levando em conta a figura de entrada como base, tendo isso em mente passamos as dimensões da imagem lida como parâmetros de criação do array, também indicamos que o tipo de dado que desejamos armazenar no array é do tipo `numpy.uint8`, já que estamos interessados apenas nos valores entre 0 e 255. O próximo passo após criar o array é localizar o centro da imagem, tornando possível criar um intervalo que divide a imagem em partes iguais.
{% highlight python %}

for i in range(0,meiov):
    for j in range(meioh,len(img[0])):
        img2[i,j] = img[meiov + i,j - meioh]

{% endhighlight %}

Por fim mapeamos os valores da imagem original em seus quadrantes inversos, este primeiro conjunto de laços de repetição faz a captura dos valores que ficarão no primeiro quadrante da imagem (canto superior direito), os demais laços operam de forma semelhante, mudando apenas as regiões de interesse. Com isso conseguimos o seguinte resultado:

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/regioes.png)
*Figura 3. Exemplo de saída do programa trocaregioes.py*

Neste segundo exemplo vimos o quão simples é utilizar a biblioteca Numpy juntamente com o OpenCV para manipulação de imagens. Com isso encerro esse primeiro post da série de processamento digital de imagens, bons estudos e até uma próxima!