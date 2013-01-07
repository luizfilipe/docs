## Controles

Uma das grandes vantagens do console OUYA é que os jogadores usam um controle *real*! O controle do OUYA tem:
- quatro botões digitais (O, U, Y, e A)
- um direcional de quatro direções (D-Pad)
- dois joysticks analógicos (LS, RS)
- dois botões digitais que são ativados quando os joysticks são pressionados para baixo (L3, R3)
- dois botões "de ombro" digitais (L1, R1)
- dois gatilhos analógicos (L2, R2)
- um touchpad, configurado para agir como um mouse. (Touchpad)

**Nota:** Os gatilhos analógicos também agem como botões digitais, com um princípio em 0,5 para o valor analógico.

Como as interfaces para os controles são tão cruciais, nós fizemos um bom trabalho para fazer sua vida mais fácil.

##### Constantes

A classe **OuyaController** tém constantes específicas para os botões e eixos do OUYA. Uma pequena porção pode ser vista abaixo:
```java
public static final int BUTTON_O;
public static final int BUTTON_U;
public static final int BUTTON_Y;
public static final int BUTTON_A;
```

Com essas constantes em mão, é totalmente aceitável aceitar input pelos métodos padrão **onkeyDown**, **onKeyUp**, ou **onGenericMotionEvent**.

##### Perguntar o estado a qualquer momento.

Se você quer uma flexibilidade extra de perguntar o estado do controle a qualquer moment, você pode usar o resto da classe **OuyaController**
Pra começar, isso significa que a sua atividade deve enviar as chamadas de **onKeyDown**, **onKeyUp**, ou **onGenericMotionEvent** para o **OuyaController**:

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

Uma vez que o **OuyaController** estiver manuseando os eventos, você pode pegar uma instância da classe de duas maneiras: identificação do dispositibo, ou número do jogador:

```java
OuyaController c = OuyaController.getControllerByDeviceId(deviceId);
OuyaController c = OuyaController.getControllerByPlayer(playerNum);
```
Agora é simples pra perguntar sobre os valores um botão ou eixo:

```java
float axisX = c.getAxisValue(OuyaController.AXIS_LSTICK_X);
float axisY = c.getAxisValue(OuyaController.AXIS_LSTICK_Y);
boolean buttonPressed = c.getButton(OuyaController.BUTTON_O);
```

Com isso, você pode focar em fazer um ótimo jogo ao invés de manusear input.
