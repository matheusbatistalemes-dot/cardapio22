# Sistema de pedidos

Cardápio digital com pedidos enviados pelo WhatsApp, editor de lojas e exportação de sites estáticos. O painel usa Firebase Authentication, Firestore e Cloud Storage.

## Estrutura

- `login.html`, `planos.html`: entrada e apresentação do plano.
- `admin/`: editor, prévia, backup e exportação de lojas.
- `cardapio.html?loja=slug`: cardápio público carregado do Firestore.
- `js/firebase-init.js`: inicializa o Firebase sem carregar autenticação na página pública.
- `index.html`, `config/`, `css/`, `js/app.js`: modelo de site estático que o editor exporta para cada cliente.
- `firestore.rules`, `storage.rules`, `firebase.json`: regras de acesso do backend Firebase.
- `scripts/admin-bundle.js`: atualiza o app incluído na prévia/exportação do admin.

## Modelo de dados

- `users/{uid}` guarda o perfil e o estado da assinatura. O próprio cliente só pode criar o perfil inicial sem plano; não pode alterar plano, vencimento ou bloqueio.
- `lojas/{slug}` guarda os dados privados de edição e o `ownerUid`. O painel consulta apenas lojas da conta autenticada.
- `cardapios/{slug}` guarda somente a configuração pública consumida pelo cardápio.
- As imagens ficam em `users/{uid}/lojas/{slug}/...` no Cloud Storage. O painel publica URLs no cardápio depois de enviar as imagens.

Ao excluir uma loja, o painel remove os documentos privados e públicos. Os arquivos de imagem não são apagados pelo navegador, porque ZIPs já publicados podem continuar usando essas URLs; remoção definitiva precisa de uma rotina confiável no backend.

O cardápio público pode ler um documento ativo de `cardapios`. A edição, listagem, gravação e exclusão são limitadas pelas regras do Firebase. Não use regras abertas como `allow read, write: if true`.

## Desenvolvimento local

Sirva a pasta `_` por HTTP, por exemplo:

```sh
python3 -m http.server 5500
```

Abra `http://localhost:5500/login.html` para autenticação e `http://localhost:5500/cardapio.html?loja=slug` para um cardápio publicado. O Firebase precisa permitir o domínio local de autenticação.

## Regras Firebase

O projeto Firebase configurado no cliente é `sistema-de-pedidos-b1416`. Com o Firebase CLI instalado e autenticado, a partir desta pasta, publique as regras:

```sh
firebase deploy --project sistema-de-pedidos-b1416 --only firestore:rules,storage
```

As regras exigem plano ativo para ler e editar lojas privadas; mantêm cinco dias de tolerância após o vencimento. O dono pode excluir a própria loja. O estado do plano só deve ser atualizado por um backend confiável que confirme o pagamento.

Documentos antigos em `lojas` sem `ownerUid` não serão exibidos pelas regras novas. Antes de publicar regras em uma base que já contenha dados, migre cada loja para o dono correto e crie sua configuração pública em `cardapios`. Não atribua ownership pelo navegador.

## Assinaturas

O login Google e o cadastro/login por email criam o perfil inicial sem plano. O checkout e a confirmação automática de pagamento ainda não estão conectados; por isso, o botão de assinatura não libera acesso. A próxima etapa para cobrar clientes é criar um backend de checkout/webhook que altere `users/{uid}` usando credenciais de servidor.

## Exportação de sites estáticos

O editor pode gerar um ZIP independente para uma loja. O ZIP usa `admin/app-files.js`, que é gerado a partir de `index.html`, `css/app.css` e `js/app.js`. Depois de mudar esses arquivos, rode:

```sh
node scripts/admin-bundle.js
```

Cada site estático exportado mantém sua configuração local em `config/loja.js` e envia pedidos diretamente para o WhatsApp da loja.

## Documentos legais e privacidade

`privacidade.html`, `termos.html`, `cancelamento.html` e `cookies.html` são rascunhos. Não trate esses textos como publicação final: dados do fornecedor, canal de privacidade, configurações de produção, retenção e revisão jurídica ainda precisam ser confirmados. O aceite de termos não está registrado no cadastro. A lógica de pagamento/assinatura não foi alterada nesta revisão.

O cardápio exportado inclui `privacidade-loja.html`, que explica o envio voluntário do pedido pelo WhatsApp e deixa identificados os dados legais que cada lojista precisa completar. O código do cardápio não deve persistir nome, endereço ou observações pessoais no armazenamento local.