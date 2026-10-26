```
public class LanaDelRey extends AdvancedRobot {

  private int direcaoMovimento = 1; 
  private double energiaAnteriorInimigo = 100.0; 

@Override
public void run() {
     setBodyColor(Color.BLACK);       
     setGunColor(Color.RED);         
     setRadarColor(Color.YELLOW);
     setBulletColor(Color.ORANGE);  
     setScanColor(Color.CYAN); 

     setAdjustGunForRobotTurn(true);
     setAdjustRadarForGunTurn(true);
     setAdjustRadarForRobotTurn(true); 

     while (true) {
         turnRadarRightRadians(Double.POSITIVE_INFINITY);
     }
}
```

Bom agora entramos na configuração básica do nosso robô aqui e onde criamos nossas variáveis que depois vão ser usadas e usamos o método #run do #AdvancedRobot .

As variáveis são para a direção de movimento do nosso robô, ou seja, se ele vai para frente ou para trás, sendo que 1 ele vai para frente é -1 ele vai para trás, é para detectar quando nosso inimigo atirar com base da energia dele, de forma simples o inimigo começa com 100 de vida e se ele perder vida ente 0,1 a 3,0 quer dizer que ele deu um tiro em nosso direção(a regra padrão do #robocode é que a força do tiro pode ir até 3,0), assim ajudando a gente prever quando devemos desviar ou não.

A biblioteca do #AdvancedRobot  dá métodos já pré-programados e daí onde vem o #run.
o #run é o método principal de execução(parecido com o main de outras linguagens, mas sendo um loop contínuo durante a batalha).

O método #run  serve para 2 coisas fundamentais:
	1. Configuração inicial (Ex: definição de cores, ajuste em parâmetros e configuração do comportamento de cada parte do robô logo no início da rodada).
	2.  Loop Principal de Ações Contínuas (Ex: Controlar o que o robô faz por padrão quando não está respondendo a eventos específicos, como ser escaneado ou bater na parede, no nosso código, ele serve para manter o radar girando de forma infinita à procura do inimigo).

Nosso método #run tem um estrutura específica onde primeiro ajustamos a cor do nosso robô, depois fazemos cada parte do nosso robô ser independente, ou seja, cada parte do nosso robô pode virar para lados diferentes e agir de formas diferentes, se tornando entidades independentes.
Esta parte é vital pois sem isso nosso robô seria obsoleto e teria dificuldades muito maiores para atirar, mirar é se movimentar.

O loop infinito #while é o que manda o radar girar sem parar para a direita.(Double.POSITIVE_INFINITY funciona como um comando para girar eternamente o #scanner).
Ele fica no método #run pois assim que a rodada começar, o robô já precisa achar o inimigo. Este comando faz o radar varrer o campo até encontrar o adversário. Assim que o inimigo é encontrado, o #robocode  interrompe temporariamente esse loop e dispara o evento #onScannedRobot(citado em [[Código Principal Da Estratégia do nosso robô]] ).

Tradução literal de algumas partes deste código:
```
setAdjustGunForRobotTurn(true)
```
**Tradução literal:** _Definir ajuste da arma para a rotação do robô (como verdadeiro)._
**Na prática:** Desacopla o canhão para que ele _não_ gire junto quando o corpo do robô virar.

```
setAdjustRadarForGunTurn(true)
```
**Tradução literal:** _Definir ajuste do radar para a rotação da arma (como verdadeiro)._
**Na prática:** Desacopla o radar para que ele _não_ gire junto quando o canhão se mover.

```
setAdjustRadarForRobotTurn(true)
```
**Tradução literal:** _Definir ajuste do radar para a rotação do robô (como verdadeiro)._
**Na prática:** Desacopla o radar para que ele _não_ gire junto quando o corpo do robô se mover.