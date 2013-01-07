## Compras dentro do aplicativo

Compras dentro do aplicativo (In-App Purchasing -IAP) � como seu app pode fazer dinheiro.  O OUYA Developer Kit (ODK) � designado para ser tanto f�cil quanto seguro.

#### Defini��es

**Produto** -- Isso � o que � comprado pelo usu�rio.  Ele tem tr�s atributos:
* *Identificador* -- um identificador mostrado para o desenvolvedor (�nico por desenvolvedor)
* *Pre�o* -- O custo do produto
* *Nome* -- o nome mostrado para o usu�rio

**Compr�vel** -- identifica um produto quando fazendo compras.

**Recibo** -- Informa��o sobre uma compra pr�via. Ele tem tr�s atributos:
* *ID do Produto* -- o identificador do produto
* *Pre�o* -- a quantia que foi paga
* *Data da compra* -- quando a compra foi feita

**Direitos** --  um produto que pode ser comprado apenas uma vez e contiua dispon�vel ao jogo mesmo ap�s uma reinstala��o

**Consum�vel** -- um produto que pode ser comprado v�rias vezes

**Assinatura** -- um produto que ir� ser comprado automaticamente num certo agendamento [n�o implementado ainda]

#### Iniciando a ODK

Toda funcionalidade do IAP passa pelo objeto **OuyaFacade**.  Um desses objetos deve ser criado no come�o da aplica��o e ser usada para todos os pedidos de IAP.
```java
	// Seu identificador de desenvolvedor (developer id) pode ser encontrado no Portal de Desenvolvedores
	public static final String DEVELOPER_ID = "00000000-0000-0000-0000-000000000000";

	// Cria um OuyaFacade
	private OuyaFacade OuyaFacade = new OuyaFacade();

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		OuyaFacade.init(this, DEVELOPER_ID);
		super.onCreate(savedInstanceState);
	}
```
Claro, quando seu aplicativo terminar, � bom informar o **OuyaFacade** sobre isso:
```java
	@Override
	protected void onDestroy() {
		OuyaFacade.shutdown();
		super.onDestroy();
	}
```
Agora estamos prontos pra mais a��o!

#### Criando produtos
Para os usu�rios te darem dinheiro, voc� precisa primeiro criar produtos pra eles comprarem. Isso � feito pelo website do OUYA via [Portal de Desenvolvedores](https://devs.ouya.tv/developers).
Depois de fazer o login, clique no menu **Produtos** e depois no link **Novo Produto**. Isso vai levar voc� at� a p�gina que voc� pode entrar o ID do produto, pre�o e nome.

#### Pegando informa��es do produto

Uma vez que os produtos forem criados, voc� pode querer que seu aplicativo pergunte ao servidor os atuais nomes e pre�os. Isso � feito pedindo uma lista de produtos, onde produtos que voc� estiver interessado podem ser especificiados pela identifica��o do produto (Product ID).
```java
	// Essa � a lista de identifica��es dos produtos que nosso app sabe sobre.
	public static final List<Purchasable> PRODUCT_ID_LIST =
		Arrays.asList(new Purchasable("sharp_sword"));
```
Como esse pedido tem que percorrer pela internet para os servidores do OUYA, a resposta vai ser retornada por um mecanismo de callback. Cabe ao seu aplicativo criar um objeto ouvinte apropriado (listener). Ouvintes podem extender **OuyaResponseListener** e terem tr�s m�todos de callback:

* **onSuccess**	-- o pedido foi sucedido e os dados s�o passados
* **onFailure**	-- o pedido falhou e o erro � passado
* **onCancel**	-- o usu�rio cancelou o pedido (por exemplo, ele apertou "Cancelar" quando solicitado a senha)

Agora iremos criar nosso pr�prio ouvinte! Nesse exemplo, n�s estamos extendendo a **CancelIgnoringOuyaResponseListener** que ignora canelamentos. Dessa maneira, n�s apenas fornecemos os m�todos **onSuccess** e **onFailure**:
```java
	OuyaResponseListener<ArrayList<Product>> productListListener =
		new CancelIgnoringOuyaResponseListener<ArrayList<Product>>() {
			@Override
			public void onSuccess(ArrayList<Product> products) {
				for(Product p : products) {
					Log.d("Produto", p.getName() + " custa " + p.getPriceInCents());
				}
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Erro", errorMessage);
			}
		};
```
Ent�o, como n�s pegamos os dados que queremos? Fazendo um pedido via **OuyaFacade**:
```java
	OuyaFacade.requestProductList(PRODUCT_ID_LIST, productListListener);
```
T�o f�cil!

#### Fazendo uma compra

Uma vez que os jogadores experimentarem seu aplicativo, eles ficam super viciados e avidamente compram todos os produtos que voc� oferecer! Pra fazeer isso aconteer, n�s primeiro precisamos criar um ouvinte:
```java
	CancelIgnoringOuyaResponseListener<Product> purchaseListener =
		new CancelIgnoringOuyaResponseListener<Product>() {
			@Override
			public void onSuccess(Product product) {
				Log.d("Compra", "Parabens voce comprou: " + product.getName());
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Erro", errorMessage);
			}
		};
```
Com esse ouvinte definido, n�s fazemos a compra com apenas uma linha:
```java
	Purchasable productToBuy = PRODUCT_ID_LIST.get(0);
	OuyaFacade.requestPurchase(productToBuy, purchaseListener);
```
Agora � s� esperar o dinheiro come�ar a entrar...

#### Consultando recibos de compras

Nesse ponto, n�s podemos pegar informa��o dos produtos e compr�-los, mas e se o usu�rio comprou algo numa sess�o anterior? O ODK disponibiliza um modo para listar todos os recibos de compras. Sim, n�s precisaremos de outro objeto ouvinte!
_Por favor note que apenas produtos que s�o direitos ser�o devolvidos. Isso � para evitar a re-atribui��o de compras de consum�veis que os jogadores j� consumiram._

Por quest�es de seguran�a, os recibos s�o retornados criptografados e devem ser decifrados dentro do pr�prio aplicativo. Para ajudar com isso, voc� pode usar o m�todo **decryptReceiptResponse** do **OuyaEncryptionHelper**.

Vamos dar uma olhada no nosso ouvinte:
```java
	CancelIgnoringOuyaResponseListener<String> receiptListListener =
		new CancelIgnoringOuyaResponseListener<String>() {
			@Override
			public void onSuccess(String receiptResponse) {
				OuyaEncryptionHelper helper = new OuyaEncryptionHelper();
				List<Receipt> receipts = null;
				try {
					receipts = helper.decryptReceiptResponse(receiptResponse);
				} catch (IOException e) {
					throw new RuntimeException(e);
				}
				for (Receipt r : receipts) {
					Log.d("Recibo", "Voce comprou: " + r.getIdentifier())
				}
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Erro", errorMessage);
			}
		};
```
Como de costume, fazer um pedido � bem simples:
```java
	OuyaFacade.requestReceipts(receiptListListener);
```
O deciframento do recibo ocorre dentro do aplicativo para ajudar a previnir hackers. Movendo o deciframento pra cada aplicativo n�o h� nenhum "peda�o de c�ddigo" que um hacker possa atacar pra quebrar a criptografia de todos aplicativos. No futuro, n�s iremos encorajar desenvolvedores de usar o m�todo **decryptReceiptResponde**. Eles ir�o precisar mover o m�todo para seus pr�prios aplicativos, e *perturbar* o que ele faz superficialmente (trocar for-loops para while-loops e assim vai) para ajudar a deixar as coisas ainda mais seguras.
Atualmente, a ODK est� sobre desenvolvimento pesado, ent�o o m�todo para ajuda vai auxiliar a isolar voc� das mudan�as "de baixo do cap�".

#### Identificando o usu�rio
Se o seu aplicativo conversa com um servidor externo, � constantemente necess�rio ter um identificador �nico para o usu�rio atual (por exemplo, pra guardar uma tabela de pontua��o em um site), Isso � feito em um padr�o normal de criar um ouvinte:
```java
	CancelIgnoringOuyaResponseListener<String> gamerUuidListener =
		new CancelIgnoringOuyaResponseListener<String>() {
			@Override
			public void onSuccess(String result) {
				Log.d("UUID", "UUID is: " + result);
			}

			@Override
			public void onFailure(int errorCode, String errorMessage) {
				Log.d("Erro", errorMessage);
			}
		};
```
Ent�o fazendo um pedido:
```java
	OuyaFacade.requestGamerUuid(gamerUuidListener);
```
**Nota**: Esses UUIDs de jogo s�o diferente para cada desenvolvedor; dois aplicativos por desenvolvedores diferentes que pedirem o UUID do mesmo usu�rio v�o receber resultados diferentes.
