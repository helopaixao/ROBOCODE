Parte 1: Calculando a Potência do Tiro (`potenciaTiro`)
```
double potenciaTiro = Math.min(400 / e.getDistance(), 3.0);
```

Se alguém te perguntar o que essa linha faz, explique estes **3 pontos principais**:

1. **`400 / e.getDistance()` (A proporção com a distância):**
    
    - No #robocode , quanto mais perto o inimigo está, mais forte você pode (e deve) atirar para causar mais dano.
        
    - A divisão `400 / distância` cria uma relação **inversamente proporcional**: se o inimigo está muito perto (ex: 50 pixels), o resultado é alto ($400 / 50 = 8$). Se ele está longe (ex: 400 pixels), o resultado é baixo ($400 / 400 = 1$).
        
2. **`Math.min(..., 3.0)` (O limite máximo do jogo):**
    
    - No #robocode , **a potência máxima permitida para uma bala é 3.0** e a mínima é 0.1.
        
    - A função `Math.min` compara os dois valores e escolhe **o menor**. Ou seja, se o inimigo estiver super perto e o cálculo der `8` ou `5`, a função barra o número e força ele a ser exatamente **`3.0`**. Se o inimigo estiver longe e o cálculo der `1.2`, ela mantém `1.2`.
        
3. **Resumo prático:**
    
    - Essa linha garante que o robô atire com força máxima (3.0) quando o inimigo estiver próximo, e vá reduzindo a força do tiro conforme o inimigo se afasta.

Parte 2: Calculando a Velocidade da Bala (`velocidadeBala`)
```
double velocidadeBala = 20 - (3 * potenciaTiro);
```

Esta linha aplica uma **regra física oficial do próprio #robocode . Se te perguntarem sobre ela, explique os seguintes **2 detalhes**:

1. **A Regra de Ouro do #robocode  (Força vs. Velocidade):**
    
    - No jogo, **balas mais fortes são mais lentas, e balas mais fracos são mais rápidas**.
        
    - A fórmula padrão do #robocode para calcular isso é: $\text{Velocidade} = 20-(3 \times \text{Potência})$.
        
2. **Como a matemática funciona na prática:**
    
    - **Se o tiro for fraco (`potenciaTiro = 1.0`):**$$\text{velocidadeBala} = 20 - (3 \times 1.0) = 20 - 3 = \mathbf{17 \text{ pixels/turno}}$$
        _(A bala voa muito rápido!)_
        
    - **Se o tiro for forte (`potenciaTiro = 3.0`):**$$\text{velocidadeBala} = 20 - (3 \times 3.0) = 20 - 9 = \mathbf{11 \text{ pixels/turno}}$$
        _(A bala voa mais devagar, dando mais tempo para o inimigo desviar)._

### Por que isso importa para a Mira Preditiva?

A `velocidadeBala` que você calculou aqui é a peça-chave que alimenta o loop de predição que vimos antes. Como a bala mais forte é mais lenta, o `tempoPreviso` aumenta, fazendo o robô mirar mais longe para compensar o tempo de viagem do projétil!