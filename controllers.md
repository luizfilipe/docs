## Controles

Uma das grandes vantagens do console OUYA � que os jogadores usam um controle *real*! O controle do OUYA tem:
- quatro bot�es digitais (O, U, Y, e A)
- um direcional de quatro dire��es (D-Pad)
- dois joysticks anal�gicos (LS, RS)
- dois bot�es digitais que s�o ativados quando os joysticks s�o pressionados para baixo (L3, R3)
- dois bot�es "de ombro" digitais (L1, R1)
- dois gatilhos anal�gicos (L2, R2)
- um touchpad, configurado para agir como um mouse. (Touchpad)

**Nota:** Os gatilhos anal�gicos tamb�m agem como bot�es digitais, com um princ�pio em 0,5 para o valor anal�gico.

Como as interfaces para os controles s�o t�o cruciais, n�s fizemos um bom trabalho para fazer sua vida mais f�cil.

##### Constantes

A classe **OuyaController** t�m constantes espec�ficas para os bot�es e eixos do OUYA. Uma pequena por��o pode ser vista abaixo:
```java
public static final int BUTTON_O;
public static final int BUTTON_U;
public static final int BUTTON_Y;
public static final int BUTTON_A;
```

Com essas constantes em m�o, � totalmente aceit�vel aceitar input pelos m�todos padr�o **onkeyDown**, **onKeyUp**, ou **onGenericMotionEvent**.

##### Perguntar o estado a qualquer momento.

Se voc� quer uma flexibilidade extra de perguntar o estado do controle a qualquer moment, voc� pode usar o resto da classe **OuyaController**
Pra come�ar, isso significa que a sua atividade deve enviar as chamadas de **onKeyDown**, **onKeyUp**, ou **onGenericMotionEvent** para o **OuyaController**:

```java
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    boolean handled = OuyaController.onKeyDown(keyCode, event);
    return handled || super.onKeyDown(keyCode, event);
}

@Override
public boolean onKeyUp(int keyCode, KeyEvent event) {
    boolean handled = OuyaController.onKeyUp(keyCode, event);
    return handled || super.onKeyUp(keyCode, event);
}

@Override
public boolean onGenericMotionEvent(MotionEvent event) {
    boolean handled = OuyaController.onGenericMotionEvent(event);
    return handled || super.onGenericMotionEvent(event);
}
```

Uma vez que o **OuyaController** estiver manuseando os eventos, voc� pode pegar uma inst�ncia da classe de duas maneiras: identifica��o do dispositibo, ou n�mero do jogador:

```java
OuyaController c = OuyaController.getControllerByDeviceId(deviceId);
OuyaController c = OuyaController.getControllerByPlayer(playerNum);
```
Agora � simples pra perguntar sobre os valores um bot�o ou eixo:

```java
float axisX = c.getAxisValue(OuyaController.AXIS_LSTICK_X);
float axisY = c.getAxisValue(OuyaController.AXIS_LSTICK_Y);
boolean buttonPressed = c.getButton(OuyaController.BUTTON_O);
```

Com isso, voc� pode focar em fazer um �timo jogo ao inv�s de manusear input.
