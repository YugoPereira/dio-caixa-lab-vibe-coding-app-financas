# Product Requirements Document (PRD) - FinChat AI

## 1. Introdução e Visão do Produto
O **FinChat AI** é um gerenciador financeiro pessoal focado na experiência "Chat-First". O objetivo é reduzir o atrito do controle financeiro manual, permitindo que o usuário registre gastos e investimentos através de uma interface de conversação natural (texto e imagem), como se estivesse falando com um consultor financeiro no WhatsApp.

## 2. Instruções Técnicas (Stack & Arquitetura)

### Frontend
- **Framework:** React + Vite + TypeScript.
- **Estilização:** Tailwind CSS.
- **Componentes UI:** Shadcn UI (obrigatório para consistência visual).
- **Ícones:** Lucide-React.
- **Gráficos:** Recharts (Responsivos).

### Backend & Banco de Dados (Supabase)
O sistema deve utilizar o Supabase para Autenticação e Persistência de dados.

**Esquema do Banco de Dados (Tabelas Obrigatórias):**

1.  **`transactions`**
    - `id` (uuid, primary key)
    - `user_id` (uuid, foreign key para auth.users)
    - `amount` (numeric)
    - `type` (text: 'income' | 'expense')
    - `category` (text)
    - `description` (text)
    - `date` (date)
    - `receipt_url` (text, nullable) - Para armazenar link da imagem do recibo.
    - `created_at` (timestamp)

2.  **`investments`**
    - `id` (uuid, primary key)
    - `user_id` (uuid)
    - `name` (text)
    - `type` (text: 'fixed' | 'variable')
    - `current_value` (numeric)

3.  **`chat_messages`** (Opcional, se o chat precisar de histórico persistente além da sessão)
    - `id`, `user_id`, `role` ('user' | 'assistant'), `content`, `created_at`.

### Inteligência Artificial (Google Gemini)
- **Provedor:** Google Generative AI SDK (`@google/generative-ai`).
- **Modelo:** `gemini-1.5-flash` (Otimizado para velocidade e custo).
- **Integração:**
    - O frontend deve enviar o input do usuário (texto ou base64 da imagem) para a API.
    - O prompt do sistema deve instruir o Gemini a retornar **apenas um JSON estruturado** (sem markdown) contendo os dados da transação para serem inseridos no banco.
    - **Variável de Ambiente:** `VITE_GOOGLE_API_KEY`.

---

## 3. Funcionalidades Core (MVP)

### A. Autenticação Simples
- Login e Cadastro utilizando **Supabase Auth** (Email e Senha).
- Proteção de rotas (apenas usuários logados acessam o app).
- Implementação de RLS (Row Level Security) nas tabelas para isolamento de dados.

### B. Chat Bimodal (Texto & Visão)
- **Input de Texto:** Usuário digita "Gastei 50 reais na padaria".
- **Input de Imagem:** Usuário clica no clipe/ícone de imagem, seleciona foto de um recibo. A IA processa via OCR/Visão e extrai os dados.
- **Feedback:** O Chat deve mostrar uma mensagem amigável de confirmação ("✅ Salvo! R$ 50 em Alimentação") e não o JSON cru.

### C. Dashboard "Máquina do Tempo"
- **Filtro Global:** Um seletor de Data (Mês/Ano) no topo da tela.
- **Reatividade:** Ao mudar a data, todos os componentes abaixo devem atualizar:
    - Cards
