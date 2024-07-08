# Twilio Conversation Integration

## Descrição
Este projeto integra a funcionalidade de conversação do Twilio em uma aplicação web. Ele permite a criação e gerenciamento de conversas com usuários através da plataforma Twilio, facilitando a comunicação e interação em tempo real.

## Requisitos
- 🥎 Node.js v14 ou superior
- 🥎 Vue.js v2.6.12 ou superior
- 🥎 Twilio Node.js SDK
- 🥎 Outras dependências listadas no arquivo package.json

## Instalação
1. Clone este repositório para sua máquina local:
#### git clone https://github.com/AutoGestaoWeb/TwilioConversationIntegration.git
2. Navegue até o diretório do projeto:
#### cd TwilioConversationIntegration
3. Instale as dependências do projeto:
#### npm install
Configure as variáveis de ambiente necessárias. Crie um arquivo .env na raiz do projeto e adicione as seguintes variáveis:

#### TWILIO_ACCOUNT_SID=seu_account_sid
#### TWILIO_AUTH_TOKEN=seu_auth_token
#### TWILIO_API_KEY_SID=seu_api_token
#### TWILIO_API_KEY_SECRET=seu_api_key_secret
#### TWILIO_CONVERSATIONS_SERVICE_SID=seu_conversationsid
#### TWILIO_FLOW_SID=seu_twilio_flowsid
#### EXECUTION_TIME= 10


# Uso

Para iniciar a aplicação em ambiente de desenvolvimento, execute o comando:

### npm run serve
## A aplicação estará disponível em http://localhost:8080.

## Estrutura do Projeto
src/: Contém o código-fonte da aplicação.
components/: Componentes Vue utilizados na aplicação.
views/: Páginas da aplicação.
assets/: Arquivos estáticos (imagens, CSS, etc.).
public/: Arquivos públicos (index.html, favicon, etc.).
package.json: Lista de dependências e scripts do projeto.
Contribuição
Contribuições são bem-vindas! Para contribuir, siga os passos abaixo:

Faça um fork deste repositório.
Crie uma nova branch para sua feature ou correção de bug:
bash
Copiar código
git checkout -b minha-feature
Faça as alterações necessárias e commit:
bash
Copiar código
git commit -m "Adiciona minha nova feature"
Envie suas alterações para o seu fork:
bash
Copiar código
git push origin minha-feature
Abra um Pull Request neste repositório.
Licença
Este projeto está licenciado sob a licença MIT. Consulte o arquivo LICENSE para mais informações.

Contato
Para mais informações, entre em contato com a equipe de desenvolvimento em email@empresa.com.
