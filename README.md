<p align="center">
  <img src="src/assets/logo-text.png" alt="Logo" width="200"/>
</p>

<p align="center">
  <strong>Sistema de gerenciamento para os nossos melhores amigos</strong>
</p>

<p align="center">
  <a href="#apresentacao">Apresentação</a> •
  <a href="#sobre">Sobre</a> •
  <a href="#tecnologias">Tecnologias</a> •
  <a href="#funcionalidades">Funcionalidades</a> •
  <a href="#instalação">Instalação</a> •
  <a href="#uso">Como Usar</a> •
  <a href="#equipe">Equipe</a>
</p>

## 🎬 <a id="apresentacao"></a>Apresentação

https://github.com/user-attachments/assets/bf190e6f-1155-4088-bed2-9402ff8d0e50

[![API Backend](https://img.shields.io/badge/API%20Backend-GitHub-blue?logo=github)](https://github.com/Igorcbraz/AmigoPet-api)

## 📋 <a id="sobre"></a>Sobre

AmigoPet é uma aplicação web desenvolvida para auxiliar no gerenciamento dos nossos pets, facilitando o agendamento de consultas, controle de vacinas, cadastro de pets, além de outras funcionalidades essenciais para o dia a dia destes bichinhos.

## 🚀 <a id="tecnologias"></a>Tecnologias

Este projeto foi desenvolvido utilizando as seguintes tecnologias:

- [Vue.js](https://vuejs.org/)
- [Quasar Framework](https://quasar.dev/)
- [Node.js](https://nodejs.org/)
- [JavaScript/ES6+](https://developer.mozilla.org/en-US/docs/Web/JavaScript)

## ✨ <a id="funcionalidades"></a>Funcionalidades

- Cadastro de usuários
- Cadastro de pets
- Início (Dashboard) - Painel principal com visão geral do pet.
- Agenda - Calendário com os agendamentos (consultas, vacinas, banhos, adestramento, pet sitter).
- Notificações - Lista de lembretes: medicação, alimentação, passeios, higiene, etc.

## 🔧 <a id="instalação"></a>Instalação

### Pré-requisitos

- Node.js (versão 20.x ou superior)
- NPM (10.x ou superior) ou Yarn

### Passos para instalação

1. Clone o repositório

```bash
git clone https://github.com/Igorcbraz/AmigoPet.git
cd AmigoPet
```

2. Instale as dependências

```bash
npm install
```

## 📱 <a id="uso"></a>Como Usar

Iniciar em modo de desenvolvimento

```bash
quasar dev
```

Verificação de código (Lint)

```bash
npm run lint
```

Formatação de código

```bash
npm run format
```

Compilar para produção

```bash
quasar build
```

## ⚠️ Troubleshooting (Notificações do Navegador)

Se o botão de notificações não aparece ou você não recebe o prompt do navegador, verifique os pontos abaixo:

1. Permissão do navegador
   - Browsers modernos (Firefox, Chrome) exigem que Notification.requestPermission() seja chamado a partir de um gesto do usuário (por exemplo, um clique). A aplicação agora exibe um botão "Ativar notificações" próximo ao ícone de sino — clique nele para abrir o prompt.
   - Se você recusar a permissão, o estado permanecerá como "denied" e será necessário alterar manualmente nas configurações do navegador para reativar.

2. Testando localmente
   - Em localhost, notificações são permitidas via HTTP em desenvolvimento. Em produção, o site precisa estar servido via HTTPS para usar Notifications.

3. Verifique o Console do navegador
   - Mensagem esperada quando a permissão é solicitada fora de um gesto do usuário:
     "The Notification permission may only be requested from inside a short running user-generated event handler." — isso indica que o requestPermission foi chamado no load; atualizar a página e usar o botão resolve.

4. Fallback
   - Se o navegador não suportar a API de Notifications ou a permissão for negada, o aplicativo mostrará notificações in-app (toasts) para garantir que o usuário receba os lembretes.

5. Ainda com problemas?
   - Abra DevTools -> Console e Network (filtrar XHR) e verifique se as requisições para /api/pet-care-history e /api/pets retornam 200.
   - Caso prefira, exporte um HAR e o console logs e envie para análise.

## 👨‍💻 <a id="equipe"></a>Equipe de Desenvolvimento

- IGOR COSTA BRAZ
- FELIPE PEREIRA SANTANA
- ELVIO AMERICO RIBEIRO
- ANDRE LUIZ FERNANDES
- CAIO MOREIRA SANTOS DE OLIVEIRA

## 📄 Licença

Este projeto está sob a licença MIT - veja o arquivo LICENSE para mais detalhes.

<p align="center"> Desenvolvido com ❤️ pela equipe AmigoPet </p>
