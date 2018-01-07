Hell Triangle - Solução para o Algoritmo
=============
O Problema
-----------
Encontre a soma máxima em um caminho de cima para baixo em um triângulo. o
O movimento de um elemento para o próximo é restrito apenas a
elementos mais próximos logo abaixo do elemento anterior. Exemplo:
```
   6
  3 5
 9 7 1
4 6 8 4
```
O caminho máximo é 6 + 5 + 7 + 8 = 26.

A Solução
------------
A ideia inicial é dividir os triangulos, como o exemplo abaixo:

```
 9
4 6
```

Podemos encontrar o caminho máximo comparando 4 e 6. Então, esse triângulo
teria uma soma de `9 + max (4, 6)` que produz 15. Então, se obtivermos
um triângulo maior, como:

```
  3
 9 8
4 6 8
```

Podemos primeiro encontrar a soma nos dois subtriglares menores. Único
Na esquerda é o exemplo anterior e o da direita é
`([8], [6, 8])` qual soma máxima é `8 + max (6, 8)` que é igual a 16.
Então, podemos observar o nosso triângulo como o seguinte:

```
  3
15 16
```
Então, se optarmos por ir para a esquerda, o caminho máximo é 15, mas se nós
escolhermos o caminho da direita é 16. A escolha ideal será o caminho
que maximiza a soma total do triângulo, que neste caso é o caminho da direita. 

Então podemos reescrever isso como algo como

```
3 + max(sum_at_left + sum_at_right)
```

Primeiro, definimos nosso tipo de dados como alias para os tipos primitivos.
```
python from typing import Sequence

Row = Sequence[int]
Triangle = Sequence[Row]
```

Para obter os triângulos esquerdo e direito, definimos a função `split()` como
```
python
from typing import Tuple

def split(tri: Triangle) -> Tuple[Triangle, Triangle]:
    """Return the left and right subtriangles as (left, right)."""
    left_right_pairs = ((row[:-1], row[1:]) for row in tri[1:])
    left, right = zip(*left_right_pairs)  # Unzip pairs
    return (left, right)
```
Note-se que, se descartarmos o elemento superior do triângulo, podemos obter o triângulo esquerdo descartando o último elemento de cada linha. Da mesma forma, podemos obter o triângulo direito, descartando o primeiro elemento de cada linha.

Portanto, a função para calcular a soma máxima pode ser definida como
```
python 
def max_path(tri: Triangle) -> int:
    """Return the maximum sum on a path from top to bottom."""
    if len(tri) == 1:
        return tri[0][0]

    left, right = split(tri)
    return tri[0][0] + max(max_path(left), max_path(right))
```
Primeiro verifica se o triângulo é apenas um único elemento, um caso em que a soma máxima seria apenas ela mesma. Caso contrário, ele divide o triângulo na esquerda e direita e retorna a expressão que renderá sua soma máxima, que é seu próprio valor mais a soma máxima do subtriangle esquerdo ou o direito, o que for maior.

Esta solução é muito próxima do que fizemos à mão, mas não é muito eficiente. Foram necessários 75 segundos para obter a soma máxima de um triângulo de 25 linhas na minha máquina. Esta é uma realidade por duas razões. Primeiro, ele cria dois novos triângulos com informações repetitivas para cada chamada da função `split ()`. E também, irá recalcular muitas chamadas de função idênticas. Por exemplo, veja este triângulo:
```
     0
    0 0
   0 6 0
  0 3 5 0
 0 9 7 1 0
0 4 6 8 4 0
```
Sim, é o primeiro triângulo coberto com zeros. O triângulo interno coberto por 6 vai ter sua soma máxima calculada duas vezes por causa da adição de zeros. Isso acontece porque, na primeira divisão, temos esses dois:
```
  Left         Right
    0            0
   0 6          6 0
  0 3 5        3 5 0
 0 9 7 1      9 7 1 0
0 4 6 8 4    4 6 8 4 0
```
E pior, os subtrigulos cobertos por 3 e 5 serão recalculados 4 vezes cada, e seus subtriangles ainda mais vezes.

Bem, então vamos fazer uma função que não copia nenhuma informação e também é fácil implementar uma programação dinâmica de cache e lá.
```
python
from functools import lru_cache

def max_path_cached(tri: Triangle) -> int:
    """Return the maximum sum on a path from top to bottom."""
    last_row_index = len(tri) - 1

    @lru_cache(maxsize=None)  # Avoid repetitive calculations
    def max_path_from(i: int, j: int) -> int:
        """Return the maximum possible sum from subtriangle [i][j]."""
        if i < last_row_index:
            left = max_path_from(i + 1, j)
            right = max_path_from(i + 1, j + 1)
            return tri[i][j] + max(left, right)
        return tri[i][j]

    return max_path_from(0, 0)
```

Esta função define uma constante do `last_row_index` do triângulo, define outra função chamada `max_path_from ()` e retorna com os parâmetros `i = 0, j = 0`. Observe que esta última função tem acesso a `last_row_index` e ao parâmetro `tri`. A função `max_path_from ()` retorna a soma máxima do subtriangle que começa na linha `i` e coluna` j`. Ele funciona pelo mesmo princípio do algoritmo anterior, mas navega pelo triângulo com índices para evitar a cópia e facilitar o armazenamento em cache. Primeiro, ele vai para a esquerda no triângulo, até chegar a um último elemento de linha, que a soma é mesmo. Em seguida, ele volta um quadro de pilha e terá o valor para a variável esquerda. Em seguida, obterá o valor para o subtriangle direito, retornará o seu caminho máximo e volte novamente a um quadro de pilha. E assim por diante. Isto e mais corresponderiam a algo como:

```
  3          3          3        19
 9 8   =>  15 8   =>  15 16  =>  
4 6 8        6 8
```

Todas essas funções estão definidas no arquivo `hell_triangle.py`.

Testes
-----
Existem testes para as funções `hell_triangle.py` no arquivo `hell_triangle_test.py`. Os testes foram planejados com `pytest`, mas como ele não usa nenhuma função auxiliar de `pytest`, ele pode ser executado sem `pytest`.

Você também pode usar `mypy` ou outro verificador estático para testar a digitação do código.
