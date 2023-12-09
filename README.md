Conversão entre codificações UTF-8 e varint

O objetivo deste trabalho é implementar, na linguagem C, duas funções (utf2varint e varint2utf), que convertem, respectivamente, arquivos em formato UTF-8 para o formato varint e arquivos em formato varint para o formato UTF-8 


Instruções Gerais
Leia com atenção o enunciado do trabalho e as instruções para a entrega. Em caso de dúvidas, não invente. Pergunte!
O trabalho deve ser entregue até meia-noite (23:59) do dia 20 de outubro.
Trabalhos entregues com atraso perderão um ponto por dia de atraso.
Trabalhos que não compilem (isto é, que não produzam um executável) não serão considerados! Ou seja, receberão grau zero.
Os trabalhos podem ser feitos em grupos de no máximo dois alunos.
Alguns grupos poderão ser chamados para apresentações orais / demonstrações dos trabalhos entregues.

Codificação Unicode
Em computação, caracteres de texto são tipicamente representados por códigos especificados por algum padrão de codificação. Um padrão bastante conhecido é a codificação ASCII, que utiliza valores inteiros de 0 a 127 para representar letras, dígitos e alguns outros símbolos. Algumas extensões dessa codificação utilizam também a faixa de valores de 128 a 255 para representar, por exemplo, caracteres acentuados e alguns outros símbolos adicionais.

A codificação ASCII e outros padrões de codificação que se limitem a um único byte para representar caracteres ficam restritos a apenas 256 símbolos diferentes. A codificação Unicode foi criada, no final da década de 1980, para permitir a representação de um conjunto maior de símbolos. A versão corrente dessa codificação é capaz de representar os caracteres utilizados por todos os idiomas conhecidos, além de diversos outros símbolos.

Cada caractere em Unicode é associado a um código (code point) na faixa de 0 a 0x10FFFF, o que, com a exclusão de alguns códigos não utilizados, permite a representação de 1.111.998 símbolos diferentes. Na notação adotada pelo padrão Unicode, U+xxxx identifica o código com valor hexadecimal xxxx. Por exemplo, o código U+00A9 (o código do símbolo ©) corresponde ao valor hexadecimal 0x00A9.

Uma faixa de valores (o intervalo 0xD800 a 0xDFFF) não deve ser utilizada para a representação de caracteres ou símbolos. Essa faixa é reservada para uso como surrogates na codificação UTF-16, permitindo a representação de códigos superiores a 0xFFFF. Contudo, na prática essa regra é muitas vezes ignorada em outras codificações UTF, que não UTF-16.

Codepoints Unicode podem ser representados através de diferentes tipos de codificação de caracteres. O padrão Unicode define as codificações UTF-8, UTF-16 e UTF-32. Para este trabalho, a codificação de interesse é a UTF-8. Essa codificação é a mais utilizada atualmente em sistemas Linux, e é também a codificação dominante na Web. 


Codificação UTF-8

Na codificação UTF-8, os códigos dos caracteres são representados em um número variável de bytes. O tamanho mínimo utilizado para representar um caractere em UTF-8 é um byte (8 bits); se a representação necessita de mais espaço, mais bytes são utilizados (até o máximo de 4 bytes).
Uma característica importante é que a codificação UTF-8 é compatível com o padrão ASCII, ou seja, os 128 caracteres associados aos códigos de 0 a 0x7F em ASCII tem a mesma representação (em um único byte) em UTF-8.

A tabela a seguir indica para cada faixa de valores de códigos Unicode o número de bytes necessários para representá-los e a codificação usada para essa faixa.

Código Unicode	Representação UTF-8 (byte a byte)
U+0000 a U+007F	0xxxxxxx
U+0080 a U+07FF	110xxxxx 10xxxxxx
U+0800 a U+FFFF	1110xxxx 10xxxxxx 10xxxxxx
U+10000 a U+10FFFF	11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

Note que:

x é o valor de um bit. O bit x da extrema direita é o bit menos significativo.
Um código representado em um apenas 7 bits tem sempre 0 no bit mais significativo.
Se um código é representado por uma sequência de bytes, o número de 1s no início do primeiro byte indica o número total de bytes da sequência. Esses 1s são seguidos sempre por um 0.
Todos os bytes que se seguem ao primeiro começam com 10 (indicando que é um byte de continuação).
Exemplos:
O símbolo © tem código Unicode U+00A9. 
Em binário A9 é 1010 1001. Usando a codificação de 2 bytes para a faixa U+0080 a U+07FF temos:
11000010 10101001 = 0xC2 0xA9

O primeiro byte começa com 110, indicando que a sequência é composta por dois bytes. A seguir vêm os cinco primeiros bits do código Unicode (note o preenchimento com zeros à esquerda para completar a porção do código do caractere colocada no primeiro byte da sequência).

O segundo byte começa com 10, indicando que é um byte de continuação. A seguir vêm os próximos seis bits do código Unicode.

O símbolo ≠ tem código Unicode U+2260. 
Em binário 2260 é 0010 0010 0110 0000. Usando a codificação de 3 bytes para a faixa U+0800 a U+FFFF temos:
11100010 10001001 10100000 = 0xE2 0x89 0xA0

O primeiro byte começa com 1110, indicando que a sequência é composta por três bytes. A seguir vêm os quatro primeiros bits do código Unicode

O segundo e o terceiro bytes começam com 10, indicando que são bytes de continuação. Cada um deles tem, em seguida, os próximos seis bits do código Unicode.


Codificação varint
Varint é um método de serialização de inteiros que, assim como UTF-8, utiliza um número variável de bytes, de forma que quanto menor o valor a ser codificado, menor o número de bytes. Esse tipo de codificação é utilizado, por exemplo, no padrão Protocol Buffers, um mecanismo de serialização de dados definido pela Google para uso em protocolos de comunicação e armazenamento de dados.

Para fazer a codificação varint de um valor inteiro, o valor original é dividido em grupos de 7 bits, começando com os bits menos significativos. Cada um desses grupos gera um byte da codificação; o bit mais significativo de cada byte gerado indica se ele é o último byte do valor codificado (0) ou se há mais bytes em seguida (1).

Vejamos um exemplo, a codificação do inteiro 300 como um varint. Em binário, temos:

00000000 00000000 00000001 00101100
Pegamos o primeiro grupo de 7 bits (os menos significativos, em azul), e acrescentamos um oitavo bit para indicar que haverá mais bytes a seguir:

 10101100

Este é o primeiro byte da codificação. Pegamos agora o próximo grupo de 7 bits. Como os demais grupos contém apenas zeros, este será o último byte do valor codificado:
 00000010

A codificação varint do valor inteiro 300 utiliza então dois bytes:
10101100 00000010
que corresponde, em hexadecimal, à sequência 0xAC 0x02.


Funções de Conversão
O objetivo deste trabalho é implementar, na linguagem C, as funções utf2varint e varint2utf, que realizam respectivamente, a conversão de arquivos em formato UTF-8 para o formato varint e arquivos em formato varint para o formato UTF-8. 


Conversão UTF-8 para varint

A função utf2varint deve ler o conteúdo de um arquivo de entrada codificado em UTF-8, obter os valores dos codepoints representados, e gravar em um arquivo de saída esses valores codificados como varints. O protótipo (cabeçalho) dessa função é o seguinte:
  int utf2varint(FILE *arq_entrada, FILE *arq_saida);
Os dois parâmetros da função são dois arquivos abertos em modo binário: o arquivo de entrada (arq_entrada) e o arquivo de saída (arq_saida).
O valor de retorno da função utf82varint é 0, em caso de sucesso e -1, em caso de erro de E/S. Em caso de erro, a função deve emitir, na saída de erro (stderr), uma mensagem indicando qual o tipo de erro ocorrido (leitura ou gravação) e retornar imediatamente.

Por simplicidade, você pode considerar que o arquivo de entrada sempre estará CORRETAMENTE CODIFICADO. Dessa forma, você não precisa implementar o tratamento de erros de codificação do arquivo de entrada, apenas erros de E/S.



Conversão varint para UTF-8

A função varint2utf deve ler o conteúdo de um arquivo de entrada contendo valores inteiros codificados como varints, obter os valores inteiros originais, e gravar em um arquivo de saída os valores obtidos, codificados em UTF-8.
O protótipo da função é o seguinte:

  int varint2utf(FILE *arq_entrada, FILE *arq_saida);
Novamente, os parâmetros da função são dois arquivos abertos em modo binário: o arquivo de entrada (arq_entrada) e o arquivo de saída (arq_saida).
Assim como na função anterior, o valor de retorno é 0, em caso de sucesso e -1, em caso de erro de E/S. Em caso de erro, a função deve retornar após emitir, na saída de erro (stderr) uma mensagem indicando o tipo de erro ocorrido (leitura ou gravação).

Por simplicidade, você pode considerar que o arquivo de entrada sempre estará CORRETAMENTE CODIFICADO, ou seja, todos os valores inteiros codificados no arquivo correspondem a codepoints válidos.



Implementação e Execução
Você deve criar um arquivo fonte chamado converte.c contendo as duas funções descritas acima, e funções auxiliares, se for o caso. Esse arquivo não deve conter uma função main!

O arquivo converte.c deverá incluir o arquivo de cabeçalho converte.h , fornecido aqui. Além desse arquivo de cabeçalho, seu arquivo fonte somente deverá incluir arquivos de cabeçalho relativos a funções de bibliotecas padrão de C.

Para testar seu programa, crie um outro arquivo, por exemplo, teste.c, contendo uma função main. Crie seu programa executável, por exemplo teste, com a linha:

gcc -Wall -o teste converte.c teste.c

Exemplos para teste

Para testar suas funções você pode usar os arquivos fornecidos a seguir:
Arquivos pequenos: os dois arquivos contém o mesmo "conteúdo", codificado em UTF-8 e varint.
Arquivo Pequeno UTF-8
Arquivo Pequeno varint
Arquivos grandes: os dois arquivos contém o mesmo "conteúdo", codificado em UTF-8 e varint.
Arquivo UTF-8
Arquivo varint
Para testar seu programa, use técnica de TDD (Test Driven Design), na qual testes são escritos antes do código. O propósito é garantir ao desenvolvedor (você) ter um bom entendimento dos requisitos do trabalho antes de implementar o programa. Com isto a automação de testes é praticada desde o início do desenvolvimento, permitindo a elaboração e execução contínua de testes de regressão. Desta forma fortalecemos a criação de um código que nasce simples, testável e próximo aos requisitos do trabalho. Os passos gerais para seguir tal técnica:
Escrever/codificar um novo teste
Executar todos os testes criados até o momento para ver se algum falha
Escrever o código responsável por passar no novo teste inserido
Executar todos os testes criados até o momento e atestar execução com sucesso
Refatorar código testado
Importante! Se, por acaso, você não implementar alguma função, crie uma função dummy para que o programa de teste acima além do usado para a correção do trabalho possa ser gerado sem problemas.

Essa função deverá ter o nome da função não implementada e o corpo

 { return; } 
Esta técnica é denominada de mocking que consiste em criar funções apenas para teste quando as reais ainda não se encontram implementadas. Isto é também prático para o trabalho incremental em equipe, pois permite tornar indepentende o trabalho de cada membro.
Lembre-se que deve haver uma preocupação constante em subtituir as funções dummy ou Mocks pelos códigos corretos quando estes estiverem testados e disponibilizados.

Dicas
Implemente seu trabalho por partes, testando cada parte implementada antes de prosseguir.

Por exemplo, você pode implementar primeiro a leitura de um arquivo UTF-8, decodificando os caracteres para obter o valor dos codepoints correspondentes, exibindo esses valores na tela. Quando essa parte estiver funcionando, implemente a codificação varint sobre os valores, gerando um arquivo de saída.

Para verificar o conteúdo do arquivo gravado, você pode usar o utilitário hexdump. Por exemplo, o comando

hexdump -C <nome-do-arquivo>
exibe o conteúdo do arquivo especificado byte a byte, em hexadecimal (16 bytes por linha). A segunda coluna de cada linha (entre '|') exibe os caracteres ASCII correspondentes a esses bytes, se eles existirem. Experimente inspecionar os arquivos dados como exemplo com o hexdump!
Para abrir um arquivo para gravação ou leitura em formato binário, use a função

FILE *fopen(char *path, char *mode);
descrita em stdio.h. Seus argumentos são:
path: nome do arquivo a ser aberto
mode: uma string que, no nosso caso, será "rb" para abrir o arquivo para leitura em modo binário ou "wb" para abrir o arquivo para escrita em modo binário.
A letra 'b', que indica o modo binário, é ignorada em sistemas como Linux, que tratam da mesma forma arquivos de tipos texto e binário. Mas ela é necessária em outros sistemas, como Windows, que tratam de forma diferente arquivos de tipos texto e binário (interpretando/modificando, por exemplo, bytes com códigos que correspondem a caracteres de controle, como quebra de linha).
Para fazer a leitura e gravação do arquivo, pesquise as funções fwrite/fread e/ou fputc/fgetc. 
