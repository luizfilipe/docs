## Compras dentro do aplicativo

Compras dentro do aplicativo (In-App Purchasing -IAP) é como seu app pode fazer dinheiro.  O OUYA Developer Kit (ODK) é designado para ser tanto fácil quanto seguro.

#### Definições

**Produto** -- Isso é o que é comprado pelo usuário.  Ele tem três atributos:
* *Identificador* -- um identificador mostrado para o desenvolvedor (único por desenvolvedor)
* *Preço* -- O custo do produto
* *Nome* -- o nome mostrado para o usuário

**Comprável** -- identifica um produto quando fazendo compras.

**Recibo** -- Informação sobre uma compra prévia. Ele tem três atributos:
* *ID do Produto* -- o identificador do produto
* *Preço* -- a quantia que foi paga
* *Data da compra* -- quando a compra foi feita

**Direitos** --  um produto que pode ser comprado apenas uma vez e contiua disponível ao jogo mesmo após uma reinstalação

**Consumível** -- um produto que pode ser comprado várias vezes

**Assinatura** -- um produto que irá ser comprado automaticamente num certo agendamento [não implementado ainda]

#### Iniciando a ODK

Toda funcionalidade do IAP passa pelo objeto **OuyaFacade**.  Um desses objetos deve ser criado no começo da aplicação e ser usada para todos os pedidos de IAP.
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
Claro, quando seu aplicativo terminar, é bom informar o **OuyaFacade** sobre isso:
```java
	@Override
	protected void onDestroy() {
		OuyaFacade.shutdown();
		super.onDestroy();
	}
```
Agora estamos prontos pra mais ação!

#### Criando produtos
Para os usuários te darem dinheiro, você precisa primeiro criar produtos pra eles comprarem. Isso é feito pelo website do OUYA via [Portal de Desenvolvedores](https://devs.ouya.tv/developers).
Depois de fazer o login, clique no menu **Produtos** e depois no link **Novo Produto**. Isso vai levar você até a página que você pode entrar o ID do produto, preço e nome.

#### Pegando informações do produto

Uma vez que os produtos forem criados, você pode querer que seu aplicativo pergunte ao servidor os atuais nomes e preços. Isso é feito pedindo uma lista de produtos, onde produtos que você estiver interessado podem ser especificiados pela identificação do produto (Product ID).
```java
	// Essa é a lista de identificações dos produtos que nosso app sabe sobre.
	public static final List<Purchasable> PRODUCT_ID_LIST =
		Arrays.asList(new Purchasable("sharp_sword"));
```
Como esse pedido tem que percorrer pela internet para os servidores do OUYA, a resposta vai ser retornada por um mecanismo de callback. Cabe ao seu aplicativo criar um objeto ouvinte apropriado (listener). Ouvintes podem extender **OuyaResponseListener** e terem três métodos de callback:

* **onSuccess**	-- o pedido foi sucedido e os dados são passados
* **onFailure**	-- o pedido falhou e o erro é passado
* **onCancel**	-- o usuário cancelou o pedido (por exemplo, ele apertou "Cancelar" quando solicitado a senha)

Agora iremos criar nosso próprio ouvinte! Nesse exemplo, nós estamos extendendo a **CancelIgnoringOuyaResponseListener** que ignora canelamentos. Dessa maneira, nós apenas fornecemos os métodos **onSuccess** e **onFailure**:
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
Então, como nós pegamos os dados que queremos? Fazendo um pedido via **OuyaFacade**:
```java
	OuyaFacade.requestProductList(PRODUCT_ID_LIST, productListListener);
```
Tão fácil!

#### Fazendo uma compra

Uma vez que os jogadores experimentarem seu aplicativo, eles ficam super viciados e avidamente compram todos os produtos que você oferecer! Pra fazeer isso aconteer, nós primeiro precisamos criar um ouvinte:
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
Com esse ouvinte definido, nós fazemos a compra com apenas uma linha:
```java
	Purchasable productToBuy = PRODUCT_ID_LIST.get(0);
	OuyaFacade.requestPurchase(productToBuy, purchaseListener);
```
Agora é só esperar o dinheiro começar a entrar...

#### Consultando recibos de compras

Nesse ponto, nós podemos pegar informação dos produtos e comprá-los, mas e se o usuário comprou algo numa sessão anterior? O ODK disponibiliza um modo para listar todos os recibos de compras. Sim, nós precisaremos de outro objeto ouvinte!
_Por favor note que apenas produtos que são direitos serão devolvidos. Isso é para evitar a re-atribuição de compras de consumíveis que os jogadores já consumiram._

Por questões de segurança, os recibos são retornados criptografados e devem ser decifrados dentro do próprio aplicativo. Para ajudar com isso, você pode usar o método **decryptReceiptResponse** do **OuyaEncryptionHelper**.

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
Como de costume, fazer um pedido é bem simples:
```java
	OuyaFacade.requestReceipts(receiptListListener);
```
O deciframento do recibo ocorre dentro do aplicativo para ajudar a previnir hackers. Movendo o deciframento pra cada aplicativo não há nenhum "pedaço de códdigo" que um hacker possa atacar pra quebrar a criptografia de todos aplicativos. No futuro, nós iremos encorajar desenvolvedores de usar o método **decryptReceiptResponde**. Eles irão precisar mover o método para seus próprios aplicativos, e *perturbar* o que ele faz superficialmente (trocar for-loops para while-loops e assim vai) para ajudar a deixar as coisas ainda mais seguras.
Atualmente, a ODK está sobre desenvolvimento pesado, então o método para ajuda vai auxiliar a isolar você das mudanças "de baixo do capô".

#### Identificando o usuário
Se o seu aplicativo conversa com um servidor externo, é constantemente necessário ter um identificador único para o usuário atual (por exemplo, pra guardar uma tabela de pontuação em um site), Isso é feito em um padrão normal de criar um ouvinte:
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
Então fazendo um pedido:
```java
	OuyaFacade.requestGamerUuid(gamerUuidListener);
```
**Nota**: Esses UUIDs de jogo são diferente para cada desenvolvedor; dois aplicativos por desenvolvedores diferentes que pedirem o UUID do mesmo usuário vão receber resultados diferentes.
