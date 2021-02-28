---
layout: post
title: Preenchendo regiões de uma imagem com Python
---


<div class="message">
  Segundo post da série de processamento digital de imagens com Python e OpenCV.
</div>

Neste post vamos realizar um breve estudo de uma tarefa bastante comum no processamento de imagens, bem como na visão artificial, iremos realizar a contagem de objetos presentes de uma determinada cena.

## Rotulando as regiões

É possível realizar a contagem de objetos através da identificação de regiões, vamos usar o processo de rotulação, onde as regiões da imagem com características comuns irão receber um identificador comum. Neste exeperimento a imagem de entrada será <a href="https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/bolhas.png" target="_blank">*bolhas.png*</a>, sendo ela do tipo binária, de forma que cada pixel só possa assumir dois valores - 0 ou 255 - nos dizendo assim que, caso o pixel faça parte do fundo da cena, ele possui valor `0`, quando o pixel for de algum objeto da cena, ele tem valor `255`.

##### Listagem 1. rotulando.py
{% highlight python %}
import cv2 as cv
import sys
import numpy as np

img = cv.imread("imagens/bolhas.png", cv.IMREAD_GRAYSCALE)


if img is None:
    sys.exit("A imagem não foi encontrada")

width = len(img[0])
height = len(img)
print('{}x{}'.format(width, height))

px = 0
py = 0
nobjects = 0

# Busca objetos presentes
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 255:
            nobjects += 1
            px = j
            py = i
            cv.floodFill(img,None,(px,py),nobjects)

print('A figura tem {} bolhas'.format(nobjects))

cv.namedWindow("Resultado", cv.WINDOW_AUTOSIZE)
img2 = np.zeros((height, width), dtype=np.uint8)
cv.equalizeHist(img, img2)
res = np.concatenate((img,img2), axis= 1)
cv.imshow("Resultado", res)


{% endhighlight %}

## Descrição do programa rotulando.py

{% highlight python %}
import cv2 as cv
import sys
import numpy as np

img = cv.imread("imagens/bolhas.png", cv.IMREAD_GRAYSCALE)


if img is None:
    sys.exit("A imagem não foi encontrada")

width = len(img[0])
height = len(img)
print('{}x{}'.format(width, height))

{% endhighlight %}

Após importar as bibliotecas que iremos utilizar, é feita a leitura da imagem em tons de cinza,  caso a leitura tenha ocorrido corretamente nós iremos imprimir no console as dimensões da cena.

{% highlight python %}

# Busca objetos presentes
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 255:
            nobjects += 1
            px = j
            py = i
            cv.floodFill(img,None,(px,py),nobjects)

print('A figura tem {} bolhas'.format(nobjects))
{% endhighlight %}

A segunda parte do programa faz a varredura na imagem, partindo do ponto (0, 0). Sempre que um pixel com valor `255` é encontrado significa que chegamos em um novo objeto, devemos agora localizar todos os vizinhos do pixel que possuem a mesma tonalidade que ele, trocando seu valor pelo rótulo correspondente do objeto, quem dita o valor a ser salvo é a variável `nobjects`, sempre que encontramos um novo objeto ela tem seu valor incrementeado, fazendo com que cada objeto tenha uma tonalidade de cinza diferente, sendo `1` a primeira tonalidade aplicada. O processo de preenchimento é feito pela função `opencv.floodFill()`, esta função  recebe como parâmetros: a imagem que iremos modificar; uma máscara de operação, neste caso não iremos utilizar uma máscara em específico; o ponto de início do procedimento; o valor que será armazenado nos pixels do objeto. Após localizar e preencher todas as regiões o programa imprime no console a quantidade de objetos encontrados.

{% highlight python %}

cv.namedWindow("Resultado", cv.WINDOW_AUTOSIZE)
img2 = np.zeros((height, width), dtype=np.uint8)
cv.equalizeHist(img, img2)
res = np.concatenate((img,img2), axis= 1)
cv.imshow("Resultado", res)

{% endhighlight %}

Após calcular a quantidade de objetos presentes na cena, queremos apresentar os resultados. A função `equalizeHist()` tem o papel de melhorar a visualização do resultado, esta função realiza operações matemáticas para normalizar o brilho e aumentar o contraste de uma imagem passada como parâmetro, iremos salvar o resultado da função em `img2`. Por fim utilizamos a função `concatenate()` da biblioteca *Numpy* para unir as cenas em uma só figura antes de serem apresentadas, o parâmetro `axis` informa se a concatenação será feita no sentido vertical ("0") ou horizontal ("1"). Ao executar o programa temos os seguintes resultados:

`256x256
A figura tem 32 bolhas `

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/objetos.png)
*Figura 1. Exemplo de saída do programa rotulando.py*

O leitor atento pode observar que o trecho de código que realiza o preenchimento das regiões poderá apresentar um mal comportamento nos casos que existem mais de 255 objetos na cena, isto ocorre pois a variável que conta a quantidade de elementos na cena também é responsável por dizer o nível de cinza a ser aplicado, como nosso intervalo de valores é **[0,255]**, valores superiores não seriam representados. Uma possível solução para o problema seria limitar a contagem até o valor máximo de 255, imprimindo uma mensagem de erro e encerrando o programa nos casos que o valor limite é superado:
{% highlight python %}

for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 255:
            nobjects += 1
            if(nobjects > 255):
                sys.exit("A cena possui mais de 255 objetos")
            px = j
            py = i
            cv.floodFill(img,None,(px,py),nobjects)

{% endhighlight %}

Caso se deseje elaborar uma versão que possibilite a contagem de mais de 255 objetos, uma opção é aumentar a quantidade de canais da cena, ampliando assim a quantidade de valores possívels para tonalidades.


## Trocando regiões

Agora vamos tentar realizar um segundo procedimento, nosso objetivo é trocar os pixels de região, de modo que os valores que estavam armazenados no quadrante superior esquerdo troquem de posição com os pixels do quadrante inferior direito, de forma semelhante, faremos a troca de valores entre os quadrantes superior direito e inferior esquerdo. Neste segundo experimento não será necessário fazer a leitura de valores de entrada adicionais, apenas da própria imagem que desejamos realizar o processamento.

<a id="listagem2"></a>
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

Aqui criamos um array utilizando a função `zeros()` da biblioteca Numpy, a ideia é criar uma nova imagem que possua as regiões trocadas, levando em conta a figura de entrada como base, tendo isso em mente passamos as dimensões da imagem lida como parâmetros de criação do array, também indicamos que o tipo de dado que desejamos armazenar no array é o `numpy.uint8`, já que estamos interessados apenas nos valores entre 0 e 255. O próximo passo após criar o array é localizar o centro da imagem, tornando possível criar intervalos que dividem a imagem em partes iguais.
{% highlight python %}

for i in range(0,meiov):
    for j in range(meioh,len(img[0])):
        img2[i,j] = img[meiov + i,j - meioh]

{% endhighlight %}

Por fim, mapeamos os valores da imagem original em seus quadrantes inversos, este primeiro conjunto de laços de repetição faz a captura dos valores que ficarão no primeiro quadrante da imagem (canto superior direito), os demais laços da <a href="#listagem2">Listagem 2</a> operam de forma semelhante, mudando apenas as regiões de interesse. Com isso conseguimos o seguinte resultado:

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/regioes.png)
*Figura 3. Exemplo de saída do programa trocaregioes.py*

Neste segundo exemplo vimos o quão simples é utilizar a biblioteca Numpy juntamente com o OpenCV para manipulação de imagens. Com isso encerro esse primeiro post da série de processamento digital de imagens, bons estudos e até uma próxima!