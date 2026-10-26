1. O Método de Colisão com a Parede (`onHitWall`)
```
@Override
public void onHitWall(HitWallEvent e) {
    direcaoMovimento = -direcaoMovimento;
    setAhead(100 * direcaoMovimento);
}
```

Se alguém te perguntar o que acontece aqui, explique estes **3 pontos principais**:

1. **O que é o `onHitWall`?**
    
    - Assim como o `onScannedRobot` dispara quando você vê o inimigo, o `onHitWall` é um **evento automático do Robocode** que roda instantaneamente no momento exato em que o seu robô **colide contra a parede** da arena.
        
2. **Invertendo a direção (`direcaoMovimento = -direcaoMovimento`):**
    
    - Se o robô estava andando para frente (`1`) e bateu de cara na parede, ele troca o valor para trás (`-1`). Isso garante que ele não fique "esmurrando" a parede eternamente desperdiçando tempo de rodada.
        
3. **Dando o passo de fuga (`setAhead(100 * direcaoMovimento)`):**
    
    - Manda o robô andar 100 pixels na nova direção oposta (agora para longe da parede), tirando-o do sufoco imediatamente.
Seguindo

2. A Função Auxiliar de Distância (`Point2D_Distance`)
```
private double Point2D_Distance(double x1, double y1, double x2, double y2) {
    return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
}
```
Esta e uma função auxiliar citada em: [[Problema 5 - Passo 3]].
Lá eu explico como funciona essa função mas só pra dar mais detalhes explicarei aqui.

Se te perguntarem o que é essa função e para que ela serve, explique estes **3 detalhes**:

1. **O que significa a assinatura?**
    
    - `private`: Significa que essa função é secreta e só pode ser usada dentro deste próprio robô.
        
    - `double`: Significa que ela retorna um número decimal como resposta (a distância exata em pixels).
        
    - `double x1, double y1, double x2, double y2`: São os 4 números que ela recebe de entrada (as coordenadas de dois pontos diferentes no mapa).
        
2. **A mágica do Teorema de Pitágoras:**
    
    - Lembra da fórmula da escola? $a^2 + b^2 = c^2$.
        
    - `x2 - x1` calcula a largura do triângulo, e `Math.pow(..., 2)` eleva ao quadrado.
        
    - `y2 - y1` calcula a altura do triângulo, e também eleva ao quadrado.
        
    - `Math.sqrt(...)` tira a raiz quadrada da soma de ambos, revelando a **hipotenusa (a distância em linha reta)** entre o ponto 1 e o ponto 2.
        
3. **Por que ela existe separada?**
    
    - Para organizar o código! Em vez de reescrever toda essa fórmula de Pitágoras gigante lá dentro do loop de predição, o programador criou essa ferramenta auxiliar para chamá-la sempre que precisar medir distâncias.