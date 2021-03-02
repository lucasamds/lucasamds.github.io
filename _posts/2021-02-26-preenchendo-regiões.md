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
<a id="trecho"></a>
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

A segunda parte do programa faz a varredura na imagem, partindo do ponto **(0, 0)**. Sempre que um pixel com valor `255` é encontrado significa que chegamos em um novo objeto, devemos agora localizar todos os vizinhos do pixel que possuem a mesma tonalidade que ele, trocando seu valor pelo rótulo correspondente do objeto, quem dita o valor a ser salvo é a variável `nobjects`, sempre que encontramos um novo objeto ela tem seu valor incrementeado, fazendo com que cada objeto tenha uma tonalidade de cinza diferente, sendo `1` a primeira tonalidade aplicada. O processo de preenchimento é feito pela função `floodFill()`, esta função do *OpenCV* recebe como parâmetros: a imagem que iremos modificar; uma máscara de operação, neste caso não iremos utilizar uma máscara em específico; o ponto de início do procedimento; o valor que será armazenado nos pixels do objeto. Após localizar e preencher todas as regiões o programa imprime no console a quantidade de objetos encontrados.

{% highlight python %}

cv.namedWindow("Resultado", cv.WINDOW_AUTOSIZE)
img2 = np.zeros((height, width), dtype=np.uint8)
cv.equalizeHist(img, img2)
res = np.concatenate((img,img2), axis= 1)
cv.imshow("Resultado", res)

{% endhighlight %}

Depois calcular a quantidade de objetos presentes na cena, queremos apresentar os resultados. A função `equalizeHist()` tem o papel de melhorar a visualização do resultado, esta função realiza operações matemáticas para normalizar o brilho e aumentar o contraste de uma imagem passada como parâmetro, iremos salvar o resultado da função em `img2`. Por fim utilizamos a função `concatenate()` da biblioteca *Numpy* para unir as cenas em uma só figura antes de serem apresentadas, o parâmetro `axis` informa se a concatenação será feita no sentido vertical ("0") ou horizontal ("1"). Ao executar o programa temos os seguintes resultados:

```
256x256
A figura tem 32 bolhas 
``` 

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/objetos.png)
*Figura 1. Exemplo de saída do programa rotulando.py*

O leitor atento pode observar que o <a href="#trecho">trecho</a> de código que realiza o preenchimento das regiões poderá apresentar um mal comportamento nos casos que existem mais de 255 objetos na cena, isto ocorre pois a variável que conta a quantidade de elementos na cena também é responsável por dizer o nível de cinza a ser aplicado, como nosso intervalo de valores é **[0,255]**, valores superiores não seriam representados. Uma possível solução para o problema seria limitar a contagem até o valor máximo de 255, imprimindo uma mensagem de erro e encerrando o programa nos casos que a quantidade limite é superado:
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

Caso se deseje elaborar uma versão que possibilite a contagem de mais de 255 objetos, uma opção é aumentar a quantidade de canais da cena, ampliando assim o número de valores possíveis para tonalidades.


## Diferenciando objetos

Neste segundo experimento, vamos um pouco mais além da contagem de objetos da cena, queremos agora diferenciar as regiões de <a href="https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/bolhas.png" target="_blank">*bolhas*</a> que possuem ou não buracos internos. Para este estudo vamos desconsiderar os objetos que se encontram nas bordas da cena, pois não temos como saber se eles possuem ou não furos internos.

<a id="listagem2"></a>
##### Listagem 2. furos.py
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

nobjects = 0

# Mudando o valor das bordas para 0
for i in range(0, width):
    if img[0][i] == 255:
        cv.floodFill(img, None, (i, 0), 0)
    if img[height - 1][i] == 255:
        cv.floodFill(img, None, (i, height-1), 0)

for i in range(0, height):
    if img[i][0] == 255:
        cv.floodFill(img, None, (0, i), 0)
    if img[i][width - 1] == 255:
        cv.floodFill(img, None, (width-1, i), 0)

img2 = img.copy()

# Busca objetos presentes
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 255:
            nobjects += 1
            cv.floodFill(img, None, (j, i), nobjects)

print('A figura tem {} formas que não tocam nas bordas'.format(nobjects))

img2 = np.concatenate((img2, img), axis=1)

cv.floodFill(img, None, (0, 0), 255)

img3 = img.copy()

comFuro = 0
semFuro = 0

# Contar objetos com e sem furos
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 0:
            comFuro += 1
            cv.floodFill(img, None, (j, i), 255)
            cv.floodFill(img, None, (j-1, i), 255)

semFuro = nobjects - comFuro

print('A imagem possui {} objetos com furos e {} sem furos.'.format(comFuro, semFuro))

img3 = np.concatenate((img3,img), axis=1)
img2 = np.concatenate((img2, img3), axis=0)
cv.putText(img2,'1', (15, 15), cv.FONT_HERSHEY_COMPLEX, 0.5, (255, 255, 255))
cv.putText(img2,'2', (271, 15), cv.FONT_HERSHEY_COMPLEX, 0.5, (255, 255, 255))
cv.putText(img2,'3', (15, 271), cv.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 0))
cv.putText(img2,'4', (271, 271), cv.FONT_HERSHEY_COMPLEX, 0.5, (0, 0, 0))

cv.namedWindow("Imagem", cv.WINDOW_AUTOSIZE)
cv.imshow("Imagem", img2)
cv.waitKey()
cv.destroyAllWindows()

{% endhighlight %}

## Descrição do programa furos.py

A lógica utilizada para resolver o problema é a seguinte: primeiro vamos remover os objetos que tocam na borda da cena; o segundo passo é realizar a contagem das regiões restantes; o terceiro passo é tornar o fundo da imagem branco; por fim identificar as que possuem furos internos.

{% highlight python %}

# Mudando o valor das bordas para 0
for i in range(0, width):
    if img[0][i] == 255:
        cv.floodFill(img, None, (i, 0), 0)
    if img[height - 1][i] == 255:
        cv.floodFill(img, None, (i, height-1), 0)

for i in range(0, height):
    if img[i][0] == 255:
        cv.floodFill(img, None, (0, i), 0)
    if img[i][width - 1] == 255:
        cv.floodFill(img, None, (width-1, i), 0)
{% endhighlight %}

Aqui, ao encontrar um pixel com valor `255` em alguma das bordas da cena, preenchemos a região a qual ele pertence com `0` utilizando a função `floodFill()`, "removendo-os" da imagem.

{% highlight python %}

# Busca objetos presentes
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 255:
            nobjects += 1
            cv.floodFill(img, None, (j, i), nobjects)

print('A figura tem {} formas que não tocam nas bordas'.format(nobjects))

{% endhighlight %}

Neste segundo segmento temos a contagem das regiões restantes, é neles que iremos buscar por objetos com furos, o resultado é apresentado no console.

{% highlight python %}

cv.floodFill(img, None, (0, 0), 255)

{% endhighlight %}

Nesta linha nós fazemos com que o fundo da cena se torne branco ("255"), é importante notar que essa mudança ocorre apenas nas regiões ao redor dos objetos da cena. Com essa troca garantimos que os únicos pontos da cena que ainda possuem cor preta são as regiões internas dos objetos, é com base nessa afirmação que iremos identificar figuras com furo.

{% highlight python %}

# Contar objetos com e sem furos
for i in range(0, height):
    for j in range(0, width):
        if img[i][j] == 0:
            comFuro += 1
            cv.floodFill(img, None, (j, i), 255)
            cv.floodFill(img, None, (j-1, i), 255)

semFuro = nobjects - comFuro

{% endhighlight %}

Nesta terceira etapa, ao encontrar um pixel preto, significa que mais uma região com furo foi encontrada, sendo assim incrementamos nosso contador de objetos com furos, preenchemos tanto a região preta como a região que continha o furo com a cor branca, "removendo-os" da cena para evitar que interfiram no resto da contagem. Ao final do processo teremos a quantidade total de objetos com furos, consequentemente, a quantidade de figuras sem furos será a diferença entre o total de objetos que não tocam nas bordas e os objetos com furo.

Ao decorrer da <a href="#listagem2">Listagem 2</a> irão ocorrer algumas cópias e concatenações de imagens para gerar o resultado final, dentre as funções utilizadas está a `putText()`, ela serve para escrever sobre uma imagem, os parâmetros passados em ordem são: imagem que será feita a escrita; texto que deve ser escrito; coordenadas do canto inferior esquerdo do texto; fonte que deve ser utilizada; escala da fonte; cor da fonte.

Ao executar o programa temos os seguintes resultados:

```
256x256
A figura tem 21 formas que não tocam nas bordas
A imagem possui 7 objetos com furos e 14 sem furos.
```

![](https://raw.githubusercontent.com/lucasamds/lucasamds.github.io/main/public/images/furos.png)
*Figura 2. Exemplo de saída do programa furos.py*

Os experimentos acima mostram um pouco do poder da função `floodFill()` e de como podemos utilizá-la para realizar contagens de objetos presentes em uma imagem. Com isso concluímos este segundo post da série de processamento digital de imagens, até a próxima!