Para uso.

```
import robocode.*;
import robocode.util.Utils;
import java.awt.Color;

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

        double meuX = getX(); 
        double meuY = getY(); 

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

    @Override
    public void onHitWall(HitWallEvent e) {
        direcaoMovimento = -direcaoMovimento;
        setAhead(100 * direcaoMovimento);
    }

    private double Point2D_Distance(double x1, double y1, double x2, double y2) {
        return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
    }
}
```