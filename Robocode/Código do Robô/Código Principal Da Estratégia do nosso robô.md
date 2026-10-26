```
@Override
public void onScannedRobot(ScannedRobotEvent e) {

    double anguloAbsolutoInimigo = getHeadingRadians() + e.getBearingRadians();

    double giroRadar = Utils.normalRelativeAngle(anguloAbsolutoInimigo - getRadarHeadingRadians());

     double extraOvershoot = Math.signum(giroRadar) * Math.toRadians(20);

     setTurnRadarRightRadians(giroRadar + extraOvershoot);

     double variacaoEnergia = energiaAnteriorInimigo - e.getEnergy();

 if (variacaoEnergia > 0.1 && variacaoEnergia <= 3.0) {
      if (Math.random() > 0.8) {
	     direcaoMovimento = -direcaoMovimento;
      }
 }
 
	 energiaAnteriorInimigo = e.getEnergy();
	 
	 setTurnRightRadians(Utils.normalRelativeAngle(e.getBearingRadians() + Math.PI / 2 - (0.5 * direcaoMovimento)));

    setAhead(100 * direcaoMovimento);
	double potenciaTiro = Math.min(400 / e.getDistance(), 3.0);
	double velocidadeBala = 20 - (3 * potenciaTiro);

    double meuX = getX(); //pega o x do nosso robo
	double meuY = getY(); //pega o y do nosso robo

    double inimigoX = meuX + e.getDistance() * Math.sin(anguloAbsolutoInimigo);
    double inimigoY = meuY + e.getDistance() * Math.cos(anguloAbsolutoInimigo);

    double orientacaoInimigo = e.getHeadingRadians();
	double velocidadeInimigo = e.getVelocity();

    double tempoPreviso = 0;

    double previsaoX = inimigoX;
    double previsaoY = inimigoY;

	for (int i = 0; i < 10; i++) {
	
	    tempoPreviso = Point2D_Distance(meuX, meuY, previsaoX, previsaoY) / velocidadeBala;
	
	    previsaoX = inimigoX + Math.sin(orientacaoInimigo) * velocidadeInimigo * tempoPreviso;
	
	    previsaoY = inimigoY + Math.cos(orientacaoInimigo) * velocidadeInimigo * tempoPreviso;
	
	}

   double anguloMira = Math.atan2(previsaoX - meuX, previsaoY - meuY);

   setTurnGunRightRadians(Utils.normalRelativeAngle(anguloMira - getGunHeadingRadians()));

	if (getGunHeat() == 0 && Math.abs(getGunTurnRemaining()) < 3) {
		setFire(potenciaTiro);
	}

}
```

Chegamos a parte mais difícil do nosso robô a #estratégia.
Nossa #estratégia principal e perseguir nossos inimigos não de forma a qual nos coloque em perigo mas sim de forma estratégica para conseguirmos dar tiros mais fortes e ao mesmo tempo desviar rapidamente de ataques se defendendo ao mesmo tempo que atacamos.

Para começar temos que compreender o método #onScannedRobot : 
Este #onScannedRobot é um método que vem da biblioteca #AdvancedRobot onde sobrescrevendo seus métodos fazemos funcionalidades e #estratégia para o nosso robô.
Este método não e chamado manualmente, pois o próprio jogo roda ele de forma automática e contínua o tempo todo.

Sempre que o nosso radar tocar ou passar por cima do robô inimigo, o #robocode avisa que achou alguém ele pausa o que o robô esta fazendo e em microssegundos, joga as informações do inimigo dentro da variável que chamamos de #e (que é um objeto do tipo ScannedRobotEvent) e executa o código que está dentro deste método.
Entendo de forma clara seria assim:
- **`@Override`**: Significa que você está sobrescrevendo/substituindo um método padrão que já existe lá na classe principal do Robocode (`AdvancedRobot`).
    
- **`public`**: Significa que o método é público (o Robocode precisa conseguir enxergá-lo de fora para executá-lo).
    
- **`void`**: Significa que o método **não retorna nenhum valor** (ele apenas executa ações e faz cálculos, sem precisar "devolver" uma resposta matemática final).
    
- **`onScannedRobot`**: É o nome exato e obrigatório da função no Robocode para quando um robô é escaneado. Se você errar uma letra maiúscula ou minúscula, o jogo não reconhece.
    
- **`(ScannedRobotEvent e)`**: É o pacote de dados (o evento) que o Robocode te entrega de bandeja. Dentro desse `e`, você consegue puxar várias informações cruciais sobre o inimigo usando métodos próprios dele:
    
    - `e.getDistance()`: Quão longe o inimigo está (em pixels).
        
    - `e.getBearingRadians()`: Qual o ângulo relativo entre a frente do seu robô e o inimigo.
        
    - `e.getEnergy()`: Quanta energia (vida) o inimigo tem naquele momento.
        
    - `e.getVelocity()`: A velocidade com que o inimigo está andando.
        
    - `e.getHeadingRadians()`: Para onde o inimigo está apontando/andando.

Bom por ser um código longo decidi separar por problemas ou seja situações onde o código soluciona um problema.

##### Problema 1: "Onde o inimigo está no mapa global, e não apenas em relação ao meu robô?"
**O Problema:** O Robocode te dá o ângulo do inimigo baseado apenas no seu robô (`e.getBearingRadians()`). Se você girar, esse número muda tudo. Como saber a direção exata dele no mapa inteiro?
##### A Solução no Código:
```
double anguloAbsolutoInimigo = getHeadingRadians() + e.getBearingRadians();
```
**Como resolve:** Pega para onde o seu robô está apontando (`getHeadingRadians()`) e soma com o ângulo relativo onde o inimigo apareceu (`e.getBearingRadians()`). Pronto! Agora você tem o **ângulo absoluto** dele no mapa.(Traduzindo: os 2 seria algo como pegar/obter a orientação em radianos)

##### Problema 2: "Como eu mantenho meu radar travado nele sem que ele escape?"
**O Problema:** Quando o radar acha o inimigo, ele precisa girar para apontar diretamente para ele. Mas se ele girar _exatamente_ em cima do inimigo, o jogo roda em frames (etapas). Se o inimigo se mover no próximo frame, o radar pode ficar "atrasado" e perder o alvo de vista.
##### A Solução no Código:
```
double giroRadar = Utils.normalRelativeAngle(anguloAbsolutoInimigo - getRadarHeadingRadians());
```

**Tradução literal:** _duplo giro do radar = Utils.normalizar ângulo relativo (ângulo absoluto do inimigo - obter radianos de rumo do radar)._

**O que cada termo significa:**

- `anguloAbsolutoInimigo`: Para onde o inimigo está apontado no mapa global.
    
- `getRadarHeadingRadians()`: Para onde o _seu_ radar está olhando neste exato microssegundo.
    
- Subtrair um pelo outro nos dá a **diferença** (quanto o radar precisa girar).
    
- `Utils.normalRelativeAngle(...)`: É uma ferramenta pronta do Robocode. Ela serve para o radar **nunca dar voltas desnecessárias**. Por exemplo, se para chegar no inimigo for mais curto girar -10 graus para a esquerda do que dar uma volta inteira de 350 graus para a direita, essa função acha o caminho mais curto.

```
double extraOvershoot = Math.signum(giroRadar) * Math.toRadians(20);
```

**Tradução literal:** _duplo extra ultrapassagem = Math.assinar(giro do radar) * Math.paraRadianos(20);_

**O que cada termo significa:**

- **`Math.signum(giroRadar)`**: Significa _"Signal Number"_ (Número de Sinal). Essa função **não importa qual é o número**, ela olha apenas se o número é **positivo (`+1`)**, **negativo (`-1`)** ou **zero (`0`)**. Ela serve para descobrir qual é o **lado** para onde o radar vai girar (horário ou anti-horário).
    
- **`Math.toRadians(20)`**: Converte o número `20` (que está em graus normais que a gente conhece) para **radianos** (que é o formato de medida matemática que o Robocode usa internamente para giros).
    
- **O resultado da multiplicação (`*`):** Se o radar precisa girar para a direita, o `Math.signum` dá `+1`. Multiplicado por 20 graus, vira `+20` graus. Se for para a esquerda, dá `-1`, virando `-20` graus. Ou seja, ele garante que os 20 graus extras vão puxar para o **mesmo lado** para onde o radar já estava indo.

```
setTurnRadarRightRadians(giroRadar + extraOvershoot);
```

**Tradução literal:** _definir para girar o radar à direita em radianos (giro do radar + extra ultrapassagem)._

**O que significa na prática:** O robô pega o quanto ele precisava girar para alcançar o inimigo (`giroRadar`) e **soma mais 20 graus** (`extraOvershoot`). O radar dá uma "esticada" a mais além do inimigo, garantindo que mesmo se o adversário for muito rápido, ele continuará dentro do feixe de luz do scanner.

##### Problema 3: "Como eu percebo se o inimigo atirou em mim para tentar desviar?"
**O Problema:** Você não tem um alarme dizendo "Cuidado, uma bala foi disparada!". Como descobrir isso programaticamente?
##### A Solução no Código:
```
double variacaoEnergia = energiaAnteriorInimigo - e.getEnergy(); 
if (variacaoEnergia > 0.1 && variacaoEnergia <= 3.0) {
    if (Math.random() > 0.8) {
        direcaoMovimento = -direcaoMovimento;
    }
}
energiaAnteriorInimigo = e.getEnergy();
```

**Como resolve:** O código compara a energia que o inimigo tinha antes com a energia que ele tem agora. No #robocode, quando um robô gasta energia entre `0.1` e `3.0`, significa que ele **disparou um tiro**. Assim que o robô detecta o tiro alheio, ele joga um dado de probabilidade (`Math.random() > 0.8`) e, se der sorte, inverte a direção em que está andando (`direcaoMovimento = -direcaoMovimento`), fazendo a bala inimiga passar direto.

##### Problema 4: "Como eu me movimento ao redor do inimigo (órbita) para desviar dos tiros dele?"
**O Problema:** Se o seu robô ficar parado, ele vira um alvo fácil. Se ele correr em linha reta para cima do inimigo, ele vai colidir. Como fazer o robô andar de lado, circulando o adversário constantemente?
##### A Solução no Código:
```
setTurnRightRadians(Utils.normalRelativeAngle(e.getBearingRadians() + Math.PI / 2 - (0.5 * direcaoMovimento)));
setAhead(100 * direcaoMovimento);
```

#### Tradução Literal e Análise Parte por Parte:

1. **`e.getBearingRadians()`**: Pega o ângulo atual onde o inimigo está em relação à frente do seu robô.
    
2. **`+ Math.PI / 2`**:
    
    - _Tradução:_ `Math.PI` vale 180 graus. Dividido por 2 (`Math.PI / 2`), dá exatamente **90 graus** em radianos.
        
    - _O que faz:_ Em vez de apontar _direto_ para o inimigo (0 graus), ele soma 90 graus. Isso faz o robô ficar **perpendicular (de lado)** em relação ao inimigo.
        
3. **`- (0.5 * direcaoMovimento)`**: Adiciona um pequeno ajuste na curva para manter o robô orbitando de forma fluida para a frente ou para trás.
    
4. **`setAhead(100 * direcaoMovimento)`**:
    
    - _Tradução:_ _definir para ir em frente (100 vezes direção do movimento)._
        
    - _O que faz:_ Manda o robô andar 100 pixels para frente (`1`) ou para trás (`-1`), combinado com a curva de 90 graus, resultando em um **movimento circular constante (órbita)** ao redor do adversário.

##### Problema 5: "Como eu calculo exatamente onde o inimigo _vai estar_ quando a minha bala chegar lá?"
**O Problema:** A bala não se teletransporta; ela viaja a uma certa velocidade pelo mapa. Se você atirar em cima de onde o inimigo está agora, ele já terá saído do lugar. Como prever o futuro?
Usamos A MIRA PREDITIVA!!
##### A Solução no Código:
1. Calculando a velocidade da bala:
```
double potenciaTiro = Math.min(400 / e.getDistance(), 3.0);
double velocidadeBala = 20 - (3 * potenciaTiro);
```
_Tradução:_ A potência do tiro é inversamente proporcional à distância (perto = forte, longe = fraco, com limite máximo de 3.0). A `velocidadeBala` física do #robocode  é calculada pela fórmula padrão: `20 - (3 * potenciaTiro)`. Quanto mais forte o tiro, mais lenta a bala se move.
Explicação detalha desta parte pois e um pouco mais difícil: [[Problema 5 - Passo 1]].

2. Convertendo distância em coordenadas do mapa (`X` e `Y`):
```
double meuX = getX(); 
double meuY = getY(); 
double inimigoX = meuX + e.getDistance() * Math.sin(anguloAbsolutoInimigo);
double inimigoY = meuY + e.getDistance() * Math.cos(anguloAbsolutoInimigo);
```

_O que faz:_ Transforma o "radar" (distância e ângulo) em um ponto fixo no plano cartesiano do mapa (`inimigoX` e `inimigoY`) usando seno e cosseno.
Explicação detalha desta parte pois e um pouco mais difícil: [[Problema 5 - Passo 2]].

3. O Loop de Predição (A máquina do tempo interativa):
```
double orientacaoInimigo = e.getHeadingRadians();
double velocidadeInimigo = e.getVelocity();

double tempoPreviso = 0;
double previsaoX = inimigoX;
double previsaoY = inimigoY;

for (int i = 0; i < 10; i++) {
    tempoPreviso = Point2D_Distance(meuX, meuY, previsaoX, previsaoY) / velocidadeBala;
    previsaoX = inimigoX + Math.sin(orientacaoInimigo) * velocidadeInimigo * tempoPreviso;
    previsaoY = inimigoY + Math.cos(orientacaoInimigo) * velocidadeInimigo * tempoPreviso;
}
```

_O que faz:_ Roda um loop de 10 tentativas (`for (int i = 0; i < 10; i++)`). A cada volta, ele calcula o tempo que a bala levaria para chegar até a posição prevista (`distância / velocidadeBala`) e projeta para onde o inimigo vai se mover nesse tempo (`velocidade * tempo`). Repetir isso 10 vezes afina o cálculo até encontrar o **ponto de encontro perfeito**.
Explicação detalha desta parte pois e um pouco mais difícil: [[Problema 5 - Passo 3]].

4. Apontar o canhão e atirar:
```
double anguloMira = Math.atan2(previsaoX - meuX, previsaoY - meuY);
setTurnGunRightRadians(Utils.normalRelativeAngle(anguloMira - getGunHeadingRadians()));

if (getGunHeat() == 0 && Math.abs(getGunTurnRemaining()) < 3) {
    setFire(potenciaTiro);
}
```

_O que faz:_

- `Math.atan2`: Descobre o ângulo exato em radianos entre o seu canhão e a coordenada futura (`previsaoX` e `previsaoY`).
    
- `setTurnGunRightRadians`: Vira o canhão para esse ponto.
    
- `if (getGunHeat() == 0 && ...)`: Verifica se o canhão está "frio" (pronto para atirar, sem calor acumulado) e se já está apontado quase perfeitamente para o alvo (margem menor que 3 graus). Se estiver, dispara!

Explicação detalha desta parte pois e um pouco mais difícil: [[Problema 5 - Passo 4]].


TERMINAMOS AQUI ALELUIA.