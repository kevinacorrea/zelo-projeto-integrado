# Zelo — projeto integrado

Pacote consolidado do frontend **Idoso 23.10**, da **Lista de Casa** e do backend **Spring Boot**. Os projetos continuam separados para serem executados no VS Code e no IntelliJ, respectivamente; a integração usa HTTP local e sessão de navegador.

## Estrutura

- `frontend-vscode/`: site estático Idoso 23.10, assistente Zelo, tela de login/cadastro e loja Lista de Casa.
- `backend-intellij/`: aplicação Spring Boot/Maven, APIs, persistência H2 local e serviços existentes.
- `backend-intellij/docs/`: documentação complementar do Web Push e do assistente.

## Rodar o backend no IntelliJ

1. Abra a pasta `backend-intellij` no IntelliJ como projeto Maven. Use **Java 21**.
2. Aguarde a importação das dependências.
3. Em **Run → Edit Configurations**, defina `SPRING_PROFILES_ACTIVE=local` na variável de ambiente. Para usar respostas generativas, inclua também `ASSISTENTE_IA_CHAVE=sua_chave_nova`; essa chave é opcional para inicializar o sistema. Não a salve no Git nem no frontend.
4. Execute `SimpleAuthApplication`. A API ficará em `http://localhost:8080`. O perfil `local` usa H2 em arquivo (`backend-intellij/connecthub-local.mv.db`), não exige MySQL e preserva contas, vínculos e tokens lembrados após reiniciar. Esse arquivo não deve ser enviado ao Git.

Também é possível iniciar pelo terminal dentro de `backend-intellij`:

```bash
SPRING_PROFILES_ACTIVE=local ./mvnw spring-boot:run
```

## Rodar o frontend no VS Code

1. Abra `frontend-vscode` no VS Code e instale a extensão **Live Server**.
2. Inicie `paginainicial/paginainicial.html` com **Open with Live Server** (não use `file://`). Normalmente o endereço será `http://127.0.0.1:5500`.
3. Com o backend ligado, escolha **Entrar → Idoso/Familiar** para cadastrar uma conta. O cadastro usa `/login/register` e cria a sessão em `/login`. Para contas existentes, use **Já tem conta? Entre por aqui**.
4. Depois de entrar, o assistente pode consultar `/assistente/status` e enviar mensagens para `/assistente/conversar`. As APIs autenticadas usam cookie de sessão e CORS com credenciais. A opção **Manter conectado** guarda apenas um token aleatório revogável em cookie `HttpOnly`, por até um ano, sem salvar a senha no navegador. Vem marcada por padrão; desmarque em aparelho compartilhado e use **Sair** para revogar o acesso do aparelho atual.
5. No painel do idoso, abra **Compras → Lista de Casa**. A loja permanece numa página própria para manter o seu CSS e JavaScript isolados. **Voltar ao Zelo** retorna ao painel. A lista e os áudios são armazenados no navegador, sem sincronização com o banco.

## Configurações da conta

Os painéis de idoso e familiar possuem o link **Configurações da conta**. A tela compartilhada permite alterar foto (imagem reduzida para JPEG), nome, e-mail, telefone e senha; ajustar tamanho do texto e alto contraste neste aparelho; ativar notificações; encerrar acessos lembrados em todos os dispositivos; e excluir a conta após confirmar a senha atual. Alterar o e-mail é imediato e o projeto não inclui verificação por e-mail. A exclusão é permanente.

Se o Live Server estiver configurado em outra porta, o backend aceita origens locais `localhost` e `127.0.0.1`. A URL base fica em `frontend-vscode/assets/js/api-config.js` (`window.ZELO_API_BASE_URL`).

## Respostas da IA e formatação

As respostas do assistente são renderizadas com um processador Markdown local, sem dependência de CDN. São suportados títulos, parágrafos e quebras de linha, **negrito**, *itálico*, ~~tachado~~, listas numeradas e com marcadores, citações, links, imagens HTTPS, tabelas, código em linha/bloco e separadores. HTML arbitrário vindo da IA é escapado antes da renderização; links só são ativados para protocolos seguros. As mensagens do usuário continuam sendo exibidas como texto simples.

Sem `ASSISTENTE_IA_CHAVE`, o backend continua iniciando e oferece o modo local acolhedor, com respostas limitadas. Para obter respostas generativas e consulta do clima, configure uma chave Groq válida apenas no ambiente do backend. O arquivo de propriedades recebido continha uma chave literal; ela foi removida. **Se a chave removida era real, revogue-a e gere outra**, pois já esteve em um arquivo compartilhado.

## Notas de integração

- O frontend do assistente envia histórico anterior ao turno atual; a mensagem atual é enviada uma única vez.
- O QR de vínculo foi alinhado ao formato emitido pelo backend (`idosoId`) e às rotas `/api/qrcode/{id}` e `/api/vinculos/qrcode`.
- O perfil Spring `local` usa cookies `SameSite=Lax` e `Secure=false` para desenvolvimento HTTP local; não use essa configuração em produção.
- A loja possui destino de WhatsApp já configurado no `frontend-vscode/loja/app.js`. Confirme que o número pertence ao contato/familiar autorizado antes de uso real.
- Login/contas usam os endpoints fornecidos pelo backend. Não se deve colocar senhas ou chaves de API em código público.
- Senhas novas são armazenadas com BCrypt; contas antigas em texto puro são migradas automaticamente para hash no primeiro login válido. O cadastro exige senha com pelo menos 8 caracteres, incluindo letra e número; para idosos, também valida e salva idade entre 1 e 120 anos. O e-mail é normalizado e comparado sem diferenciar maiúsculas/minúsculas.
- O token de dispositivo lembrado fica salvo como hash no banco e renova sua validade enquanto for usado; o logout revoga o token do aparelho. A disponibilidade através de reinicializações/deploy depende de banco persistente: o H2 local agora é em arquivo; em produção, configure PostgreSQL persistente.
- Mantenha HTTPS e cookies seguros em produção. Contas legadas com uma senha fraca continuam podendo autenticá-la para compatibilidade até a pessoa trocá-la; a migração do armazenamento para BCrypt ocorre automaticamente.
