# 📚 DeepSeek-R1 - Guia Completo de Instalação e Uso

> **Versão:** 2.0  
> **Data:** Dezembro 2024  
> **Licença:** MIT (uso comercial permitido)

---

## 📋 Índice

1. [Introdução](#-introdução)
2. [Arquitetura DeepSeek-V3](#-arquitetura-deepseek-v3)
3. [Modelos Disponíveis](#-modelos-disponíveis)
4. [Requisitos de Sistema](#-requisitos-de-sistema)
5. [Métodos de Instalação](#-métodos-de-instalação)
   - [Usando Ollama (Recomendado para Iniciantes)](#1-usando-ollama-recomendado-para-iniciantes)
   - [Usando vLLM (Recomendado para Produção)](#2-usando-vllm-recomendado-para-produção)
   - [Usando SGLang](#3-usando-sglang)
   - [Usando LMDeploy](#4-usando-lmdeploy)
   - [Usando TensorRT-LLM](#5-usando-tensorrt-llm)
   - [Via API DeepSeek](#6-via-api-deepseek)
6. [Uso Online (Sem Instalação)](#-uso-online-sem-instalação)
7. [Configurações Recomendadas](#-configurações-recomendadas)
8. [Exemplos de Uso](#-exemplos-de-uso)
9. [Preços da API](#-preços-da-api)
10. [Troubleshooting](#-troubleshooting)
11. [Recursos Adicionais](#-recursos-adicionais)

---

## 🎯 Introdução

O **DeepSeek-R1** é um modelo de raciocínio de primeira geração desenvolvido pela DeepSeek-AI, uma empresa chinesa de inteligência artificial. O modelo foi treinado usando **Reinforcement Learning (RL) em larga escala** e demonstra desempenho comparável ao OpenAI-o1 em tarefas de matemática, código e raciocínio.

### Características Principais

| Característica | Descrição |
|----------------|-----------|
| 🧠 **Raciocínio Avançado** | Capacidade de auto-verificação, reflexão e geração de cadeias de pensamento longas (CoT) |
| 🔓 **Open Source** | Código e pesos do modelo licenciados sob MIT |
| 💼 **Uso Comercial** | Permitido para modificações, trabalhos derivados e destilação |
| 🎓 **Destilação** | Modelos menores treinados com dados gerados pelo R1 |

### Variantes do Modelo

1. **DeepSeek-R1-Zero** - Treinado apenas com RL, sem fine-tuning supervisionado (SFT)
   - Primeiro modelo open-source a validar que capacidades de raciocínio podem ser incentivadas puramente através de RL
   - Demonstra auto-verificação, reflexão e geração de CoTs longos
   
2. **DeepSeek-R1** - Versão aprimorada com cold-start data antes do RL
   - Pipeline com 2 estágios de RL + 2 estágios de SFT
   - Resolve problemas de repetição infinita e mistura de idiomas do R1-Zero
   
3. **DeepSeek-R1-Distill** - Modelos menores destilados baseados em Qwen e Llama
   - Treinados com 800k amostras geradas pelo DeepSeek-R1
   - O modelo 32B supera o OpenAI-o1-mini em benchmarks!

---

## 🏗️ Arquitetura DeepSeek-V3

O DeepSeek-R1 é baseado no **DeepSeek-V3**, um modelo Mixture-of-Experts (MoE) de última geração:

### Especificações Técnicas

| Especificação | Valor |
|---------------|-------|
| **Parâmetros Totais** | 671B |
| **Parâmetros Ativos por Token** | 37B |
| **Tokens de Pré-treinamento** | 14.8 Trilhões |
| **Horas de GPU (Treinamento)** | 2.788M H800 GPU hours |
| **Arquitetura de Atenção** | Multi-head Latent Attention (MLA) |

### Inovações Arquiteturais

```
┌─────────────────────────────────────────────────────────────────┐
│                    DeepSeek-V3 Architecture                      │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Multi-head Latent Attention (MLA)                      │    │
│  │  - Gestão eficiente de memória de atenção               │    │
│  │  - Validado no DeepSeek-V2                              │    │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  DeepSeekMoE (Mixture of Experts)                       │    │
│  │  - 671B parâmetros totais                               │    │
│  │  - Apenas 37B ativados por token                        │    │
│  │  - Estratégia de balanceamento sem loss auxiliar        │    │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Multi-Token Prediction (MTP)                           │    │
│  │  - Objetivo de treinamento inovador                     │    │
│  │  - Permite speculative decoding na inferência           │    │
│  │  - Módulo MTP: 14B parâmetros adicionais                │    │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  FP8 Mixed Precision Training                           │    │
│  │  - Primeira validação em modelo de escala extrema       │    │
│  │  - Superação de gargalos de comunicação cross-node      │    │
│  │  - Overlap quase total de computação-comunicação        │    │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Pós-Treinamento com DeepSeek-R1

O DeepSeek-V3 incorpora destilação de conhecimento do DeepSeek-R1:
- Integra padrões de **verificação** e **reflexão** do R1
- Melhora significativamente o desempenho em raciocínio
- Mantém controle sobre estilo e comprimento das saídas

---

## 📦 Modelos Disponíveis

### Modelos Principais (Baseados no DeepSeek-V3)

| Modelo | Parâmetros | Tipo | Download |
|--------|------------|------|----------|
| DeepSeek-R1-Zero | ~671B | Base (MoE) | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Zero) |
| DeepSeek-R1 | ~671B | Chat (MoE) | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1) |
| DeepSeek-V3-Base | 671B + 14B MTP | Base | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3-Base) |
| DeepSeek-V3 | 671B + 14B MTP | Chat | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-V3) |

> ⚠️ **Nota:** O tamanho total no HuggingFace é 685B (671B Main Model + 14B MTP Module)

### Modelos Destilados (Recomendados para Uso Local)

| Modelo | Parâmetros | Base | Tamanho Aprox. | Download |
|--------|------------|------|----------------|----------|
| DeepSeek-R1-Distill-Qwen-1.5B | 1.5B | Qwen2.5-Math | ~1.1 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B) |
| DeepSeek-R1-Distill-Qwen-7B | 7B | Qwen2.5-Math | ~4.7 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-7B) |
| DeepSeek-R1-Distill-Llama-8B | 8B | Llama-3.1 | ~5 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-8B) |
| DeepSeek-R1-Distill-Qwen-14B | 14B | Qwen2.5 | ~9 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-14B) |
| DeepSeek-R1-Distill-Qwen-32B | 32B | Qwen2.5 | ~20 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B) |
| DeepSeek-R1-Distill-Llama-70B | 70B | Llama-3.3 | ~40 GB | [🤗 HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Llama-70B) |

> 💡 **Destaque:** O modelo `DeepSeek-R1-Distill-Qwen-32B` supera o OpenAI-o1-mini em diversos benchmarks, atingindo novos resultados state-of-the-art para modelos densos!

---

## 💻 Requisitos de Sistema

### Requisitos Mínimos por Modelo

| Modelo | RAM | VRAM GPU | Armazenamento |
|--------|-----|----------|---------------|
| 1.5B | 8 GB | 4 GB | 5 GB |
| 7B | 16 GB | 8 GB | 10 GB |
| 8B | 16 GB | 10 GB | 12 GB |
| 14B | 32 GB | 16 GB | 20 GB |
| 32B | 64 GB | 24-48 GB | 40 GB |
| 70B | 128 GB | 80+ GB | 80 GB |
| 671B (Full) | 512+ GB | Multi-GPU (8x H100/A100) | 1.5 TB |

### Software Necessário

#### Para vLLM (Recomendado para Produção)
- **Sistema Operacional:** Linux (Windows não suportado diretamente)
- **Python:** 3.10 - 3.13
- **CUDA:** 11.8+ (para GPU NVIDIA)
- **ROCm:** 6.0+ (para GPU AMD)

#### Para Ollama (Mais Fácil)
- **Sistema Operacional:** Windows 10/11, Linux, macOS
- **RAM:** Mínimo 8GB (recomendado 16GB+)

---

## 🔧 Métodos de Instalação

### 1. Usando Ollama (Recomendado para Iniciantes)

O **Ollama** é a forma mais fácil de rodar o DeepSeek-R1 localmente. Ele gerencia downloads, quantização e execução automaticamente.

#### Passo 1: Instalar o Ollama

**Windows:**
```powershell
# Baixe o instalador em: https://ollama.com/download
# Ou use winget:
winget install Ollama.Ollama
```

**Linux:**
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

**macOS:**
```bash
# Via Homebrew
brew install ollama
# Ou baixe em: https://ollama.com/download
```

#### Passo 2: Verificar Instalação

```bash
ollama --version
```

#### Passo 3: Baixar e Executar o DeepSeek-R1

```bash
# Modelo 1.5B (mais leve - ideal para testes)
ollama run deepseek-r1:1.5b

# Modelo 7B (equilibrado - recomendado)
ollama run deepseek-r1:7b

# Modelo 8B
ollama run deepseek-r1:8b

# Modelo 14B
ollama run deepseek-r1:14b

# Modelo 32B (alta performance)
ollama run deepseek-r1:32b

# Modelo 70B (melhor qualidade)
ollama run deepseek-r1:70b
```

> ⚠️ **Nota:** Na primeira execução, o modelo será baixado automaticamente. Isso pode levar alguns minutos dependendo da sua conexão.

#### Comandos Úteis do Ollama

```bash
# Listar modelos instalados
ollama list

# Remover um modelo
ollama rm deepseek-r1:7b

# Mostrar informações do modelo
ollama show deepseek-r1:7b

# Rodar modelo em modo servidor (API)
ollama serve

# Usar API via curl
curl http://localhost:11434/api/generate -d '{
  "model": "deepseek-r1:7b",
  "prompt": "Explique a teoria da relatividade"
}'
```

---

### 2. Usando vLLM (Recomendado para Produção)

O **vLLM** é uma biblioteca de alta performance para servir LLMs, desenvolvida originalmente no Sky Computing Lab da UC Berkeley. Agora é um projeto da PyTorch Foundation!

#### Características do vLLM

| Feature | Descrição |
|---------|-----------|
| **PagedAttention** | Gestão eficiente de memória de atenção |
| **Continuous Batching** | Batching contínuo de requisições |
| **CUDA/HIP Graph** | Execução rápida de modelos |
| **Quantização** | GPTQ, AWQ, AutoRound, INT4, INT8, FP8 |
| **Speculative Decoding** | Decodificação especulativa para acelerar |
| **Chunked Prefill** | Prefill em chunks para otimização |
| **API OpenAI-Compatible** | Drop-in replacement para OpenAI API |
| **Multi-Hardware** | NVIDIA, AMD, Intel, TPU, Arm, Huawei Ascend |

#### Pré-requisitos

- Linux (obrigatório)
- Python 3.10 - 3.13
- GPU NVIDIA com CUDA 11.8+ ou AMD com ROCm

#### Instalação do vLLM

**Método 1: Usando uv (Recomendado)**
```bash
# Instalar uv (gerenciador de ambiente Python ultra-rápido)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Criar ambiente e instalar vLLM
uv venv --python 3.12 --seed
source .venv/bin/activate
uv pip install vllm --torch-backend=auto
```

**Método 2: Usando conda + pip**
```bash
conda create -n vllm-env python=3.12 -y
conda activate vllm-env
pip install --upgrade uv
uv pip install vllm --torch-backend=auto
```

**Método 3: Via pip direto**
```bash
pip install vllm
```

**Método 4: Docker (AMD GPUs)**
```bash
docker pull rocm/vllm:latest
docker run --device=/dev/kfd --device=/dev/dri \
  -v <path/to/your/models>:/app/models \
  -it rocm/vllm:latest
```

#### Servir Modelos DeepSeek-R1-Distill

```bash
# Modelo 7B (GPU única)
vllm serve deepseek-ai/DeepSeek-R1-Distill-Qwen-7B \
    --max-model-len 16384

# Modelo 32B (2 GPUs com tensor parallelism)
vllm serve deepseek-ai/DeepSeek-R1-Distill-Qwen-32B \
    --tensor-parallel-size 2 \
    --max-model-len 32768 \
    --enforce-eager

# Modelo 70B (múltiplas GPUs)
vllm serve deepseek-ai/DeepSeek-R1-Distill-Llama-70B \
    --tensor-parallel-size 4 \
    --max-model-len 32768
```

#### Servir DeepSeek-V3/R1 Completo

Para o modelo completo de 671B, é necessário multi-node:

```bash
# Requer cluster com múltiplas máquinas
# Consulte: https://docs.vllm.ai/en/latest/serving/distributed_serving.html
vllm serve deepseek-ai/DeepSeek-V3 \
    --pipeline-parallel-size 8 \
    --tensor-parallel-size 8
```

#### Usar API OpenAI-Compatible

O servidor vLLM expõe uma API compatível com OpenAI em `http://localhost:8000`:

```python
# Instalar: pip install openai
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="empty"  # vLLM não requer API key por padrão
)

# Chat Completion
response = client.chat.completions.create(
    model="deepseek-ai/DeepSeek-R1-Distill-Qwen-7B",
    messages=[
        {"role": "user", "content": "Resolva: 2x + 5 = 15"}
    ],
    temperature=0.6
)

print(response.choices[0].message.content)
```

**Listar modelos disponíveis:**
```bash
curl http://localhost:8000/v1/models
```

#### Inferência Offline em Batch

```python
from vllm import LLM, SamplingParams

# Inicializar modelo
llm = LLM(model="deepseek-ai/DeepSeek-R1-Distill-Qwen-7B")

# Definir prompts
prompts = [
    "Qual é a capital do Brasil?",
    "Explique a teoria da evolução.",
    "Resolva: x² - 4 = 0"
]

# Configurar parâmetros de sampling
sampling_params = SamplingParams(
    temperature=0.6,
    top_p=0.95,
    max_tokens=1024
)

# Gerar
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(f"Prompt: {output.prompt}")
    print(f"Resposta: {output.outputs[0].text}\n")
```

---

### 3. Usando SGLang

**SGLang** oferece otimizações state-of-the-art para DeepSeek, incluindo MLA optimizations, DP Attention, FP8 e Torch Compile.

#### Instalação

```bash
pip install sglang
```

#### Iniciar Servidor

```bash
# Modelo 32B com 2 GPUs
python3 -m sglang.launch_server \
    --model deepseek-ai/DeepSeek-R1-Distill-Qwen-32B \
    --trust-remote-code \
    --tp 2

# DeepSeek-V3 completo (multi-node)
# Consulte: https://github.com/sgl-project/sglang/tree/main/benchmark/deepseek_v3
```

#### Recursos do SGLang

- ✅ MLA Optimizations
- ✅ DP Attention para modelos DeepSeek
- ✅ FP8 (W8A8)
- ✅ FP8 KV Cache
- ✅ Torch Compile
- ✅ Multi-node tensor parallelism
- ✅ Suporte NVIDIA e AMD GPUs
- 🔄 Multi-Token Prediction (em desenvolvimento)

---

### 4. Usando LMDeploy

**LMDeploy** é um framework flexível e de alta performance para servir LLMs.

#### Instalação

```bash
pip install lmdeploy
```

#### Uso

Consulte a documentação oficial: [LMDeploy DeepSeek-V3 Guide](https://github.com/InternLM/lmdeploy/issues/2960)

---

### 5. Usando TensorRT-LLM

**TensorRT-LLM** da NVIDIA oferece suporte ao DeepSeek-V3 com opções de precisão BF16 e INT4/INT8.

#### Recursos

- ✅ BF16
- ✅ INT4/INT8 weight-only
- 🔄 FP8 (em desenvolvimento)

#### Guia

Consulte: [TensorRT-LLM DeepSeek-V3](https://github.com/NVIDIA/TensorRT-LLM/tree/main/examples/deepseek_v3)

---

### 6. Via API DeepSeek

Se você não quer instalar nada localmente, use a API oficial do DeepSeek (compatível com OpenAI!).

#### Configuração

1. Acesse [platform.deepseek.com](https://platform.deepseek.com/)
2. Crie uma conta e obtenha sua [API Key](https://platform.deepseek.com/api_keys)
3. Use a API (100% compatível com OpenAI SDK)

#### Endpoints Disponíveis

| Endpoint | URL |
|----------|-----|
| Base URL | `https://api.deepseek.com` |
| Alternativa (v1) | `https://api.deepseek.com/v1` |

> ⚠️ **Nota:** O `/v1` na URL NÃO tem relação com a versão do modelo!

#### Modelos via API

| Modelo | Descrição |
|--------|-----------|
| `deepseek-chat` | Modo não-thinking do DeepSeek-V3.2 |
| `deepseek-reasoner` | Modo thinking do DeepSeek-V3.2 (equivalente ao R1) |

#### Exemplos de Uso

**Python (OpenAI SDK):**
```python
# pip install openai
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ.get('DEEPSEEK_API_KEY'),
    base_url="https://api.deepseek.com"
)

response = client.chat.completions.create(
    model="deepseek-chat",  # ou "deepseek-reasoner" para modo thinking
    messages=[
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello"}
    ],
    stream=False
)

print(response.choices[0].message.content)
```

**Node.js:**
```javascript
// npm install openai
import OpenAI from "openai";

const openai = new OpenAI({
    baseURL: 'https://api.deepseek.com',
    apiKey: process.env.DEEPSEEK_API_KEY,
});

async function main() {
    const completion = await openai.chat.completions.create({
        messages: [{ role: "system", content: "You are a helpful assistant." }],
        model: "deepseek-chat",
    });
    console.log(completion.choices[0].message.content);
}

main();
```

**cURL:**
```bash
curl https://api.deepseek.com/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}" \
  -d '{
    "model": "deepseek-chat",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "Hello!"}
    ],
    "stream": false
  }'
```

#### Funcionalidades da API

| Recurso | Descrição |
|---------|-----------|
| [JSON Output](https://api-docs.deepseek.com/guides/json_mode) | Forçar saída em formato JSON |
| [Tool Calls](https://api-docs.deepseek.com/guides/tool_calls) | Function calling |
| [Chat Prefix Completion](https://api-docs.deepseek.com/guides/chat_prefix_completion) | Completar a partir de prefixo (Beta) |
| [FIM Completion](https://api-docs.deepseek.com/guides/fim_completion) | Fill-in-the-middle (Beta) |

---

## 🌐 Uso Online (Sem Instalação)

Se você quer testar o modelo rapidamente sem instalar nada:

1. Acesse [chat.deepseek.com](https://chat.deepseek.com)
2. Crie uma conta gratuita
3. Ative o botão **"DeepThink"** para usar o modo de raciocínio avançado (R1)

---

## ⚙️ Configurações Recomendadas

Para obter o melhor desempenho do DeepSeek-R1, siga estas recomendações oficiais:

### Parâmetros de Geração

| Parâmetro | Valor Recomendado | Descrição |
|-----------|-------------------|-----------|
| `temperature` | 0.5 - 0.7 (ideal: **0.6**) | Evita repetições infinitas e saídas incoerentes |
| `top_p` | 0.95 | Controle de nucleus sampling |
| `max_tokens` | 32768 | Limite máximo de geração |

### Melhores Práticas

> [!IMPORTANT]
> **NÃO use system prompts!** Todas as instruções devem estar no prompt do usuário.

> [!TIP]
> Para problemas matemáticos, inclua no prompt:  
> *"Please reason step by step, and put your final answer within \boxed{}."*

> [!WARNING]
> O modelo pode pular o padrão de pensamento (outputando `<think>\n\n</think>`). Para forçar o raciocínio completo, **inicie a resposta com `<think>\n`**.

### Forcing Thinking Mode

Para garantir que o modelo sempre use o modo de raciocínio:

```python
# Force o modelo a iniciar com <think>
response = client.chat.completions.create(
    model="deepseek-reasoner",
    messages=[
        {"role": "user", "content": "Resolva: x² + 2x - 3 = 0"}
    ],
    temperature=0.6,
    # Alguns SDKs permitem prefixar a resposta:
    # prefix="<think>\n"
)
```

### Exemplo de Prompt Otimizado

```markdown
Resolva o seguinte problema passo a passo, mostrando todo seu raciocínio.
Coloque sua resposta final dentro de \boxed{}.

Problema: Um trem viaja a 80 km/h. Quanto tempo leva para percorrer 320 km?
```

---

## 📚 Exemplos de Uso

### Exemplo 1: Resolução de Problemas Matemáticos

```
Prompt: Resolva passo a passo: Se f(x) = 3x² - 2x + 1, qual é f'(x)?

Resposta esperada:
<think>
Para encontrar a derivada f'(x), vou aplicar a regra da potência:
- A derivada de ax^n é n*ax^(n-1)

Aplicando:
- 3x² → 2 * 3x^(2-1) = 6x
- -2x → 1 * (-2)x^(1-1) = -2
- 1 → 0 (constante)

Somando: f'(x) = 6x - 2
</think>

Portanto, \boxed{f'(x) = 6x - 2}
```

### Exemplo 2: Análise de Código

```
Prompt: Analise este código Python e identifique possíveis problemas:

def calcular_media(numeros):
    soma = 0
    for n in numeros:
        soma += n
    return soma / len(numeros)

Resposta: O código tem um problema potencial - divisão por zero caso 
a lista esteja vazia. Solução:

def calcular_media(numeros):
    if not numeros:
        raise ValueError("Lista vazia")
    return sum(numeros) / len(numeros)
```

### Exemplo 3: Raciocínio Lógico

```
Prompt: Se todos os gatos são mamíferos, e alguns mamíferos são animais 
domésticos, podemos concluir que todos os gatos são animais domésticos?

Resposta:
<think>
Premissa 1: Todos os gatos são mamíferos (Gatos ⊂ Mamíferos)
Premissa 2: Alguns mamíferos são animais domésticos (Mamíferos ∩ Domésticos ≠ ∅)

A premissa 2 diz que ALGUNS mamíferos são domésticos, não TODOS.
Os gatos podem ou não estar na interseção Mamíferos ∩ Domésticos.
</think>

Não, não podemos concluir isso. A premissa indica apenas que existe 
interseção entre mamíferos e animais domésticos, mas não que todos 
os subconjuntos de mamíferos (como gatos) estão nessa interseção.
```

### Exemplo 4: Upload de Arquivos (via API/Web)

```python
# Template para análise de arquivos
file_template = """
[file name]: {file_name}
[file content begin]
{file_content}
[file content end]

{question}
"""

prompt = file_template.format(
    file_name="relatorio.csv",
    file_content="nome,vendas\nJoão,100\nMaria,150\nPedro,80",
    question="Qual vendedor teve mais vendas?"
)
```

---

## 💰 Preços da API

Os preços são por **1 milhão de tokens**:

| Recurso | Descrição |
|---------|-----------|
| **Token** | Menor unidade de texto (palavra, número, pontuação) |
| **Cobrança** | Total de tokens de entrada + saída |

### Regras de Dedução

- Despesa = Número de tokens × Preço
- Saldo de bônus é usado primeiro (quando disponível)
- Preços podem variar - consulte [pricing](https://api-docs.deepseek.com/quick_start/pricing)

> 💡 **Dica:** O DeepSeek oferece preços muito competitivos comparado a concorrentes!

### Contato Comercial

- 📧 Email: api-service@deepseek.com
- 💬 Discord: [discord.gg/Tc7c45Zzu5](https://discord.gg/Tc7c45Zzu5)
- 🐦 Twitter: [@deepseek_ai](https://twitter.com/deepseek_ai)

---

## 🔍 Troubleshooting

### Problemas Comuns

#### ❌ "Out of Memory" (GPU)

**Solução 1:** Use um modelo menor
```bash
ollama run deepseek-r1:1.5b  # Modelo mais leve
```

**Solução 2:** Habilite quantização
```bash
# vLLM com quantização
vllm serve deepseek-ai/DeepSeek-R1-Distill-Qwen-7B \
    --quantization awq
```

**Solução 3:** Reduza o contexto
```bash
vllm serve model --max-model-len 8192  # Reduzir de 32768
```

#### ❌ Modelo gerando respostas incoerentes/repetitivas

**Solução:** Ajuste a temperatura:
```python
temperature = 0.6  # Valor ideal recomendado oficialmente
```

#### ❌ Modelo pulando o processo de raciocínio

**Solução:** Force o início do raciocínio incluindo o prefixo na resposta:
```python
# Seu prompt deve esperar que o modelo comece com <think>
```

#### ❌ Erro "CUDA not available" (vLLM)

**Solução:** 
1. Verifique se você tem uma GPU NVIDIA
2. Instale os drivers CUDA: [NVIDIA CUDA Toolkit](https://developer.nvidia.com/cuda-downloads)
3. Reinstale o PyTorch com suporte CUDA:
```bash
uv pip install vllm --torch-backend=cu126
```

#### ❌ "Hugging Face's Transformers has not been directly supported yet"

**Nota:** Para DeepSeek-R1/V3 completo, use vLLM, SGLang, LMDeploy ou TRT-LLM. Transformers direto ainda não é suportado.

Os modelos **Distill** funcionam normalmente com Transformers:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained(
    "deepseek-ai/DeepSeek-R1-Distill-Qwen-7B"
)
```

#### ❌ Download lento do modelo

**Soluções:** 
- Use conexão cabeada
- Desative VPN se estiver usando
- Use espelho do HuggingFace:
```bash
export HF_ENDPOINT="https://hf-mirror.com"
```

#### ❌ vLLM não funciona no Windows

**Nota:** vLLM requer Linux. Alternativas para Windows:
1. Use WSL2 (Windows Subsystem for Linux)
2. Use Ollama (tem suporte nativo a Windows)
3. Use Docker

---

## 📖 Recursos Adicionais

### Links Oficiais DeepSeek

| Recurso | Link |
|---------|------|
| 📄 Paper DeepSeek-R1 | [arxiv.org/abs/2501.12948](https://arxiv.org/abs/2501.12948) |
| 📄 Paper DeepSeek-V3 | [arxiv.org/pdf/2412.19437](https://arxiv.org/pdf/2412.19437) |
| 💻 GitHub DeepSeek-R1 | [github.com/deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1) |
| 💻 GitHub DeepSeek-V3 | [github.com/deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) |
| 🤗 HuggingFace | [huggingface.co/deepseek-ai](https://huggingface.co/deepseek-ai) |
| 💬 Chat Online | [chat.deepseek.com](https://chat.deepseek.com) |
| 🔌 API Platform | [platform.deepseek.com](https://platform.deepseek.com) |
| 📚 API Docs | [api-docs.deepseek.com](https://api-docs.deepseek.com/) |

### Links vLLM

| Recurso | Link |
|---------|------|
| 📚 Documentação | [docs.vllm.ai](https://docs.vllm.ai) |
| 💻 GitHub | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) |
| 📝 Blog | [blog.vllm.ai](https://blog.vllm.ai/) |
| 📄 Paper | [arxiv.org/abs/2309.06180](https://arxiv.org/abs/2309.06180) |
| 🗺️ Roadmap | [roadmap.vllm.ai](https://roadmap.vllm.ai) |
| 💬 Slack | [slack.vllm.ai](https://slack.vllm.ai) |
| 💬 Forum | [discuss.vllm.ai](https://discuss.vllm.ai) |

### Outros Frameworks

| Framework | GitHub | Documentação |
|-----------|--------|--------------|
| SGLang | [sgl-project/sglang](https://github.com/sgl-project/sglang) | - |
| LMDeploy | [InternLM/lmdeploy](https://github.com/InternLM/lmdeploy) | - |
| TensorRT-LLM | [NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) | - |
| Ollama | [ollama/ollama](https://github.com/ollama/ollama) | [ollama.com](https://ollama.com) |

### HuggingFace

| Recurso | Link |
|---------|------|
| 📚 Documentação | [huggingface.co/docs](https://huggingface.co/docs) |
| 🤗 Hub | [huggingface.co](https://huggingface.co) |

---

## 📊 Benchmarks de Desempenho

O DeepSeek-R1 foi avaliado com:
- **Temperature:** 0.6
- **Top-p:** 0.95
- **Max Tokens:** 32,768
- **Amostras por query:** 64

### Resultados Destacados

- **DeepSeek-R1** atinge performance comparável ao **OpenAI-o1** em math, code e reasoning
- **DeepSeek-R1-Distill-Qwen-32B** supera **OpenAI-o1-mini** em diversos benchmarks
- Novos resultados state-of-the-art para modelos densos (non-MoE)

---

## 📜 Citações

### DeepSeek-R1

```bibtex
@misc{deepseekai2025deepseekr1,
    title={DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning},
    author={DeepSeek-AI},
    year={2025},
    eprint={2501.12948},
    archivePrefix={arXiv},
    primaryClass={cs.CL},
    url={https://arxiv.org/abs/2501.12948}
}
```

### vLLM

```bibtex
@inproceedings{kwon2023efficient,
    title={Efficient Memory Management for Large Language Model Serving with PagedAttention},
    author={Woosuk Kwon and Zhuohan Li and Siyuan Zhuang and Ying Sheng and 
            Lianmin Zheng and Cody Hao Yu and Joseph E. Gonzalez and Hao Zhang and Ion Stoica},
    booktitle={Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles},
    year={2023}
}
```

---

## 📋 Licenciamento

| Modelo | Licença Base |
|--------|-------------|
| DeepSeek-R1 / R1-Zero | MIT License |
| DeepSeek-V3 | MIT License |
| Distill-Qwen-* | Apache 2.0 (Qwen) + MIT (fine-tuning) |
| Distill-Llama-8B | Llama 3.1 License |
| Distill-Llama-70B | Llama 3.3 License |
| vLLM | Apache 2.0 |

### Contato DeepSeek

- 📧 Email: service@deepseek.com
- 📧 API: api-service@deepseek.com
- 🐛 Issues: [GitHub Issues](https://github.com/deepseek-ai/DeepSeek-R1/issues)
- 💬 Discord: [discord.gg/Tc7c45Zzu5](https://discord.gg/Tc7c45Zzu5)

---

## 🎉 Conclusão

O DeepSeek-R1 representa um avanço significativo em modelos de raciocínio open-source. Com múltiplas opções de tamanho e métodos de instalação, é acessível desde usuários com hardware modesto até data centers empresariais.

### Recomendações por Caso de Uso

| Caso de Uso | Recomendação |
|-------------|--------------|
| **Iniciantes/Testes** | Ollama + `deepseek-r1:7b` |
| **Desenvolvimento Local** | vLLM + `DeepSeek-R1-Distill-Qwen-14B` |
| **Produção** | vLLM/SGLang + `DeepSeek-R1-Distill-Qwen-32B` |
| **Máxima Qualidade** | API DeepSeek + `deepseek-reasoner` |
| **Sem Instalação** | [chat.deepseek.com](https://chat.deepseek.com) |

---

*Documentação atualizada em Dezembro 2024. Verifique os repositórios oficiais para informações mais recentes.*
