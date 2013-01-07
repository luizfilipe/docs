## Instruções para instalar o OUYA ODK

##### MacOS
Baixe e instale as [ferramentas e SDK Android](http://developer.android.com/sdk/index.html) para o seu Mac ou PC, seguindo as instruções incluídas.

Abra o Android SDK Manager rodando ([informações detalhadas](http://developer.android.com/sdk/installing/adding-packages.html)):
```bash 
./android sdk
```

Instale esses pacotes:

- **Ferramentas**: Inclui ambos Android SDK e Android SDK Platform tools
- **Android 4.1 (API 16)**: Plataforma SDK
- **Android 4.0 (API 14)**: Plataforma SDK
- **Extras**: Biblioteca de suporte android.


Instale o Java Runtime se você for solicitado.

Você vai precisar adicionar alguns diretórios ao PATH. Assumindo que você deixou a pasta do SDK localizada em `~/android/android-sdk-macosx`, abra um terminal e adicione as seguintes três linhas ao seu `.bashrc`
```bash
export PATH=$PATH:~/android/android-sdk-macosx/tools
export PATH=$PATH:~/android/android-sdk-macosx/platform-tools
export ANDROID_HOME=~/android/android-sdk-macosx
```

Você pode precisar ajustar as suas entradas do `.bashrc` se você deixou a pasta do SDK localizada em outro lugar.


##### Windows
Para ser definido.

##### Eclipse
Para ser definido.

##### Desenvolvendo com o ODK
Use o Android AP Level 16 (Android 4.1 "Jelly Bean") quando desenvolver para o Console OUYA.

Para utilizar o OUYA API você vai precisar incluir `ouya-sdk.jar` nas bibliotecas do seu projeto, assim como `guava-r09.jar` e `commons-lang-2.6.jar`. Esses podem ser encontrados na pasta `libs`.

Para mais informação dos comandos disponíveis na API, por favor consulte a documentação de referência do OUYA API.

Para rodar o código de exemplo, abra o projeto na pasta `iap-sample-app` e siga as instruções no arquivo `README.txt`.

Para o seu aplicativo ou jogo ser reconhecido como criado para o OUYA, você vai precisar incluir um intent OUYA na entrada da sua atividade principal no manifest.
Use “ouya.intent.category.GAME” ou “ouya.intent.category.APP”.

```xml
<activity android:name=".GameActivity" android:label="@string/app_name">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
    <category android:name="ouya.intent.category.GAME"/>
  </intent-filter>
</activity>
```

A imagem do aplicativo mostrado no launcher é embutido dentro do próprio APK. O arquivo esperado é `res/drawable-xhdpi/ouya_icon.png` e o tamanho da imagem deve ser 732x412 para jogos ou 412x412 para aplicativos.

#### Substitutos de Hardware.

Para começar a desenvolver antes de ter acesso a um Console OUYA, você pode usar o emulador Android ou um tablet Android comum.

##### Software

O hardware do console OUYA  já inclui o OUYA launcher, mas quando utilizando um emulador ou um tablet Android, você precisa instalar o launcher manualmente. Esse arquivo é incluido no pacote do OUYA ODK.

Para instalar o launcher, rode:
```bash
adb install -r ouya-framework.apk
adb install -r ouya-launcher.apk
```
**Nota**: Se o OUYA launcher não estiver instalado, alguns recursos do ODK não irão funcionar corretamente.

##### Emulador

Se utilizando o emulador, configure o Android virtual device como a seguir:

- **Resolução**: 1920x1080 ou 1280x720, como desejar
- **Hardware Back/Home keys**: sim (você vai precisar adicionar isso aos parâmetros de hardware)
- **Suporte à D-Pad**: sim (você vai precisar adicionar isso aos parâmetros de hardware)
- **Target**: Android 4.1 - API Level 16
- **CPU/ABI**: Intel Atom x86
- **Device RAM size**: 1024

Nós recomendamos o uso do Intel Atom x86 CPU/ABI e as extensões HAXM da Intel para garantir que a performance do emulador é adequada para desenvolvimento de jogos. Se você está desenvolvendo código em baixo nível(low-level), você deve tomar nota que o dispositivo é baseado na arquitetura ARM. Dessa maneira, você deve desenvolver para a arquitetura ARM e usar um emulador AVD com CPU/ABI configurados para tal.

O console OUYA não tem botões de hardware para back ou menu, então os jogos não devem depender da presença destes. Colocando as teclas de hardwares nas propriedades do emulador esconderá a barra de navegação do Android, dessa forma o emulador pode preencher completamente os 1920x1080 ou 1280x720 da tela da mesma forma que o console OUYA faz.

**Nota**: Quando desenvolvendo com o emulador, não é possível emular totalmente os recursos e botões do controle OUYA.

##### Android Tablet

Se utilizar um tablet Android comum, nós recomendamos usar um tablet com um display usável o mais próximo o possível de 1920x1080 ou 1280x720.

**Nota**: A barra de navegação do Android vai consumir um pedaço da tela em tablets comuns, o que não será o caso do console OUYA.

O controle de jogos OUYA combina um controle comum (dois joysticks, um D-Pad, 4 botões de jogos, dois botões de ombro, e dois gatilhos) com um touchpad. Para testar, recomendamos usar o controle com fios do Xbox 360 combinado com um mouse ou touchpad para testar a interação de joystick e botões.

##### Manuseio da Resolução de Tela

O console OUYA suporta a saída de vídeo de 720p ou 1080p apenas.

Para jogos baseados em OpenGL, nós recomendamos criar um render buffer de 1920x1080 (se seu alvo for 1080p) ou 1280x720 (se seu alvo for 720p) Se isso não corresponder à resolução do seu dispositivo, a tela poderá ficar em janela ou será esticada dependendo do comportamento determinado pelo fabricante do dispositivo.

Para jogos baseados no Android UI Framework, siga as melhores práticas para desenvolvimento de aplicativos de display xhdpi-large e tvdpi-large do Android. Veja [developer.android.com](http://developer.android.com) para mais informação.
