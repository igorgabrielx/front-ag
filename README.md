# Front AG - Configurador de Algoritmo Genético

Interface web desenvolvida em Vue 3 para configuração e execução de algoritmos genéticos, com visualização em tempo real dos resultados e evolução do fitness.

## 🚀 Tecnologias

- **Vue 3** - Framework JavaScript reativo
- **Vite** - Build tool e dev server
- **Tailwind CSS v4** - Framework CSS utilitário
- **Chart.js** - Biblioteca para gráficos
- **Axios** - Cliente HTTP para requisições à API
- **Vue Router** - Roteamento para aplicações Vue

## 📋 Funcionalidades

- ✅ Configuração de parâmetros do algoritmo genético:
  - Tamanho da população
  - Número de gerações
  - Taxa de crossover (slider interativo)
  - Taxa de mutação (slider interativo)

- ✅ Execução assíncrona do algoritmo:
  - Envio de requisição para API
  - Polling automático do status da execução
  - Atualização em tempo real do status

- ✅ Visualização de resultados:
  - Gráfico de evolução do fitness máximo
  - Status da execução com indicadores visuais
  - Informações de início e fim da execução

## 🛠️ Instalação

### Desenvolvimento Local

1. Clone o repositório:
```bash
git clone https://github.com/igorgabrielx/front-ag.git
cd front-ag
```

2. Instale as dependências:
```bash
npm install
```

3. Inicie o servidor de desenvolvimento:
```bash
npm run dev
```

4. Acesse a aplicação em `http://localhost:5173`

### Docker

#### Usando Docker Compose (Recomendado)

1. Build e execute o container:
```bash
docker-compose up -d
```

2. Acesse a aplicação em `http://localhost:8080`

#### Usando Docker diretamente

1. Build da imagem:
```bash
docker build -t front-ag .
```

2. Execute o container:
```bash
docker run -d -p 8080:80 --name front-ag front-ag
```

3. Acesse a aplicação em `http://localhost:8080`

## 📦 Scripts Disponíveis

- `npm run dev` - Inicia o servidor de desenvolvimento
- `npm run build` - Gera a build de produção
- `npm run preview` - Preview da build de produção

## ⚙️ Configuração

### Parâmetros Padrão

- **Tamanho da População**: 100
- **Número de Gerações**: 400
- **Taxa de Crossover**: 0.65
- **Taxa de Mutação**: 0.008

### API Endpoints

A aplicação espera uma API rodando em `http://localhost:5000` com os seguintes endpoints:

- `POST /api/genetic/execute-async-ag` - Inicia execução assíncrona do algoritmo
- `GET /api/genetic/get-status-queue/<execution_id>` - Consulta status da execução
- `GET /api/genetic/get-all-executions` - Retorna todas as execuções para o gráfico

## 🎨 Interface

A interface foi desenvolvida com um tema escuro moderno, utilizando:
- Cores personalizadas via Tailwind CSS
- Ícones Material Symbols
- Gráficos interativos com Chart.js
- Design responsivo

## 📊 Estrutura do Projeto

```
front-ag/
├── src/
│   ├── views/
│   │   └── home.vue      # Página principal
│   ├── components/        # Componentes Vue
│   ├── router/           # Configuração de rotas
│   ├── assets/           # Assets estáticos
│   ├── style.css         # Estilos globais
│   └── main.js           # Entry point
├── public/               # Arquivos públicos
├── index.html            # HTML principal
├── vite.config.js        # Configuração do Vite
├── tailwind.config.js    # Configuração do Tailwind
└── package.json          # Dependências do projeto
```

## 🔧 Desenvolvimento

### Requisitos

- Node.js >= 20.19.0 ou >= 22.12.0
- npm ou yarn

### Estrutura de Componentes

O componente principal (`home.vue`) gerencia:
- Estado dos parâmetros do algoritmo
- Execução e polling do status
- Renderização do gráfico de fitness
- Interface de configuração

## 📝 Licença

Este projeto é privado.

## 👤 Autor

Igor Gabriel
