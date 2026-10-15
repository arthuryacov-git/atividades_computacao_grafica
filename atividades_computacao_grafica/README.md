# Atividades Práticas – Computação Gráfica

**Disciplina:** Computação Gráfica
**Integrantes:** Matheus Tayarol Roman Campos e Arthur Yacov Claudino Silvano e Silva

Este repositório reúne as três atividades práticas da disciplina, desenvolvidas em dupla com Python, OpenGL e a
biblioteca `core` fornecida nas aulas. Em cada arquivo foram alterados apenas os trechos marcados com `TODO`,
preservando a estrutura original do projeto.

## Estrutura do repositório

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── capturas/
│   ├── atividade1_sistema_solar.png
│   ├── atividade2_braco_robotico.png
│   └── atividade3_cubo_satelite.png
├── Aula_Transformacoes/alunos/   (atividade_sistema_solar.py, core/, formas.py, objeto.py, shaders.py ...)
├── Aula_GrafoCena/alunos/        (atividade_braco.py, core/, geometry/, material/, formas.py ...)
└── Aula_Texturas/alunos/         (atividade_cubo.py, core/, geometry/, material/, shaders/, images/ ...)
```

Cada pasta `alunos` é autocontida: tem tudo o que a atividade precisa para executar.

## Como executar

Crie o ambiente virtual **uma vez**, na raiz do projeto, e depois execute cada atividade **de dentro da pasta
`alunos` da respectiva aula** (isso evita o erro `ModuleNotFoundError: core`).

```bash
# Windows
py -3 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
cd Aula_Transformacoes\alunos
..\..\.venv\Scripts\python.exe atividade_sistema_solar.py

# macOS / Linux
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
cd Aula_Transformacoes/alunos
../../.venv/bin/python atividade_sistema_solar.py
```

Para as outras atividades, troque a pasta e o arquivo (`Aula_GrafoCena/alunos` → `atividade_braco.py` e
`Aula_Texturas/alunos` → `atividade_cubo.py`).

No PyCharm: abrir a pasta `alunos` da aula como projeto e usar **Run**. A tecla **ESC** fecha o programa.

**Capturas de tela:** ao apertar **P** (atividades 1 e 2) ou **S** (atividade 3), o programa salva
`capturas/resultado.png` dentro da pasta `alunos` da aula. Copie a imagem para a pasta `capturas/` da raiz e
renomeie conforme a estrutura acima.

---

## Atividade 1 – Mini Sistema Solar

**Conteúdo:** Transformações geométricas
**Arquivo:** `Aula_Transformacoes/alunos/atividade_sistema_solar.py`
**Captura:** tecla **P** → `capturas/atividade1_sistema_solar.png`


### Descrição da solução
Foram usadas apenas `make_translation`, `make_rotation_z` e `make_scale`, combinadas com o operador `@`.

| Objeto | Matriz | Leitura (direita → esquerda) |
| ------ | ------ | ---------------------------- |
| Sol | `R(anguloSol) @ S(0.25)` | escala, depois rotação própria |
| Órbita | `R(anguloOrbita) @ T(0.6, 0, 0)` | afasta do centro, depois gira em torno do Sol |
| Planeta | `Órbita @ R(anguloRotacaoPlaneta) @ S(0.1)` | escala, rotação própria, órbita |
| Lua | `Órbita @ R(anguloOrbitaLua) @ T(0.2, 0, 0) @ S(0.05)` | escala, afasta 0.2 do planeta, gira, acompanha a órbita do planeta |

- O planeta orbita o Sol com raio `0.6` e gira em torno do próprio eixo com velocidade diferente da órbita.
- A Lua reaproveita a matriz da órbita do planeta (por isso o acompanha) e orbita com raio `0.2`. Ela não herda a
  escala nem a rotação própria do planeta.
- As teclas **↑** e **↓** aumentam e diminuem a velocidade da órbita. A velocidade nunca fica negativa
  (`max(0, velocidade)`).
- Todas as velocidades são multiplicadas por `self.deltaTime`, pois são calculadas por segundo.

### Resposta da questão
*Por que a órbita utiliza `R @ T` e a rotação própria do planeta fica à direita da translação?*

Em uma expressão `A @ B @ ponto`, a matriz mais à direita é aplicada primeiro. Na órbita, `R @ T @ ponto`: a
translação afasta o objeto da origem (define o raio) e depois a rotação, que gira em torno da origem, leva o objeto
por um arco ao redor do Sol. A rotação própria fica à direita da translação (`R_órbita @ T @ R_própria @ S`) porque
precisa ser aplicada enquanto o planeta ainda está na origem, girando em torno do próprio centro. Se ficasse à
esquerda da translação, ela giraria o planeta em torno da origem (o Sol) em vez de em torno de si mesmo.

---

## Atividade 2 – Braço Robótico

**Conteúdo:** Grafo de Cena
**Arquivo:** `Aula_GrafoCena/alunos/atividade_braco.py`
**Captura:** tecla **P** → `capturas/atividade2_braco_robotico.png`


### Descrição da solução
A cena foi montada como uma hierarquia de objetos:

```
base (cinza)
└── ombro (Object3D, no topo da base)
    ├── braço laranja (1.2 de altura, deslocado 0.6 para cima)
    └── cotovelo (Object3D, na ponta do braço, y = 1.2)
        ├── antebraço azul (1.0 de altura, deslocado 0.5 para cima)
        └── garra cinza (na ponta do antebraço)
```

Controles:

| Tecla | Ação |
| ----- | ---- |
| **A / D** | gira a base (e todo o braço) em torno do eixo Y |
| **W / S** | movimenta o ombro em torno do eixo Z |
| **I / K** | movimenta o cotovelo em torno do eixo Z |

A velocidade de rotação é multiplicada por `self.deltaTime`.

### Resposta da questão
*Por que foi necessário utilizar pivôs em vez de simplesmente girar diretamente as caixas?*

A função `caixa(...)` cria a caixa centrada na origem local, então girá-la diretamente a faria girar em torno do
seu próprio centro, e não da articulação. O pivô (`Object3D` vazio) é posicionado exatamente no ponto da junta; a
caixa é filha dele e deslocada metade da altura para que a extremidade fique sobre o pivô. Girar o pivô gira o
segmento em torno da junta. Como o cotovelo, o antebraço e a garra são filhos do ombro, eles acompanham o
movimento automaticamente, que é a vantagem do grafo de cena.

---

## Atividade 3 – Cubo com Satélite Texturizado

**Conteúdo:** Texturas
**Arquivo:** `Aula_Texturas/alunos/atividade_cubo.py`
**Captura:** tecla **S** → `capturas/atividade3_cubo_satelite.png`


### Descrição da solução
- O cubo principal usa a textura `mosaico.png`.
- O satélite é um segundo cubo com `grade_uv.png` e `repeatUV = [2,2]`.
- O satélite é **filho** do cubo principal (`self.mesh.add(self.satelite)`), fica a `1.3` do centro no eixo X e
  tem escala `0.4`. A posição é definida **antes** da escala, para que a escala não reduza a distância.
- Por ser filho, o satélite acompanha a rotação do cubo principal (órbita).
- As teclas **↑** e **↓** aumentam e diminuem a velocidade de rotação do cubo principal (com `deltaTime`).
- O satélite também gira em torno do próprio eixo X a `2 rad/s`.

### Resposta da questão
*Qual é o papel das coordenadas UV e qual é o efeito de `repeatUV = [2,2]`?*

As coordenadas UV (u, v, normalmente entre 0 e 1) associam cada vértice do modelo a um ponto da imagem, definindo
como a textura é "colada" na geometria. Com `repeatUV = [2,2]` essas coordenadas são multiplicadas por 2, então a
imagem se repete 2 vezes na horizontal e 2 na vertical em cada face (4 cópias menores) em vez de aparecer uma
única vez.
