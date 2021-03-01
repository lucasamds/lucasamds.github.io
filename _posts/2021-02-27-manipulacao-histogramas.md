---
layout: post
title: Manipulação de histogramas com Python
---


<div class="message">
  Terceiro post da série de processamento digital de imagens com Python e OpenCV, desta vez vamos trabalhar com histogramas de imagens.
</div>

De forma resumida, o histograma é uma representração gráfica de uma determinada distribuição numérica, ele tem como objetivo mostrar a frequência que uma determinada amostra aparece na cena. Nestes experimentos vamos trabalhar com um vídeo, pois temos interesse em estudar a varição de histogramas entre uma cena e outra. Para tornar as coisas um pouco mais simples, faremos a leitura do vídeo considerando apenas seus tons de cinza.

## Equalizando um histograma

Neste primeiro experimento nós queremos realizar a equalização de um histograma e analisar o efeito que este processo causa no vídeo. Equalizar significa que iremos mudar a distribuição dos valores de ocorrência de um dado histograma, diminuindo assim as diferenças acentuadas da cena. Quando este efeito é aplicado no processamento de imagens, geralmente conseguimos acentuar detalhes que eram imperceptíveis na figura, já que temos uma normalização do brilho e aumento do contraste.

##### Listagem 1. equalizando.py
{% highlight python %}
import cv2 as cv
import numpy as np

histtam = 256
hrange = np.array([0,256])

#Leitura do vídeo
cap = cv.VideoCapture('videos/histograma.mp4')

width = int(cap.get(3))
height = int(cap.get(4))

histw = 184
histh = 92
binw = int(round(histw/histtam))
accummulate = False

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        #Transformando a imagem para tons de cinza
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        #Redimensionamento do vídeo
        frame = cv.resize(frame, (int(width/3), int(height/3)), interpolation=cv.INTER_AREA)

        equalizado = cv.equalizeHist(frame)
        #Processo de cálculo do histograma
        histograma = cv.calcHist([frame], [0], None, [histtam], hrange, accumulate=accummulate)
        histogramaeq = cv.calcHist([equalizado],[0], None, [histtam], hrange, accumulate=accummulate)


        cv.normalize(histograma, histograma, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)
        cv.normalize(histogramaeq, histogramaeq, alpha=0, beta=histh, norm_type=cv.NORM_MINMAX)

        #Desenhando o histograma
        for i in range(1,histtam):
            cv.line(frame, (binw*(i-1), histh - int(np.round(histograma[i-1]))), (binw*(i), histh - int(np.round(histograma[i]))), (0, 0, 0), thickness=2)
            cv.line(equalizado, (binw * (i - 1), histh - int(np.round(histogramaeq[i - 1]))),(binw * (i), histh - int(np.round(histogramaeq[i]))), (0, 0, 0), thickness=2)

        # Unindo as imagems
        res = np.vstack((frame, equalizado))

        cv.imshow('Frame', res)
        if cv.waitKey(5) & 0xFF == ord('q'):
            break
    else:
        break

cap.release()
cv.destroyAllWindows()

{% endhighlight %}

## Descrição do programa equalizando.py

{% highlight python %}
histtam = 256
hrange = np.array([0,256])

#Leitura do vídeo
cap = cv.VideoCapture('videos/histograma.mp4')

width = int(cap.get(3))
height = int(cap.get(4))

{% endhighlight %}

Inicialmente determinamos o intervalo de valores que nosso histograma irá possuir, como estamos trabalhando apenas com tons de cinza, temos que as amostras estão no intervalo **[0, 255]**. O comando `VideoCapture()` faz a abertura do arquivo de vídeo que iremos utilzar, caso queira realizar a captura através de uma câmera, basta fazer `cap = cv.VideoCapture(0)`. Após abrir o arquivo consultamos as dimensões do arquivo através método `get()`, ela vai retornar o valor de alguma propriedade do arquivo, a escolha é feita através do parâmetro passado, a largura e altura possuem os índices 3 e 4, respectivamente.

{% highlight python %}

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        #Transformando a imagem para tons de cinza
        frame = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
        #Redimensionamento do vídeo
        frame = cv.resize(frame, (int(width/3), int(height/3)), interpolation=cv.INTER_AREA)
{% endhighlight %}

Caso o arquivo de vídeo tenha sido aberto corretamente, damos início ao processamento. o método `read()` coleta, decodifica e nos retorna o próximo *frame* do vídeo, ele também retorna um valor booleano que diz se foi ou não encontrado um novo *frame*, esta saída quem irá nos dizer a hora de parar o processo, ao receber `false` saberemos que chegamos ao fim do arquivo. Daqui em diante as operações serão repetidas para cada novo *frame* lido

Originalmente o vídeo possui cores, porém como já foi dito, iremos utilizar apenas tons de cinza neste estudo, para isso utilizamos a função `cvtColor()`, ela recebe o *frame* que queremos modificar e realiza uma trabsformação baseada na *flag* passada, `cv.COLOR_BGR2GRAY` indica que queremos passar de um sistema RGB para um em tons de cinza. Uma última transformação que

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