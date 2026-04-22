# 🛡️ AIOps Node Agent

![Go Version](https://img.shields.io/badge/Go-1.21+-00ADD8?style=for-the-badge&logo=go)
![ONNX Runtime](https://img.shields.io/badge/ONNX-Runtime-005CED?style=for-the-badge&logo=onnx)
![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow?style=for-the-badge)

> **Trabalho de Conclusão de Curso (TCC)** - Engenharia de Software / Ciência da Computação.

Um daemon (*background agent*) ultraleve escrito em Go para monitoramento de infraestrutura e detecção de anomalias preditivas em tempo real no nível do sistema operacional.

## 📋 Sobre o Projeto

Aplicações tradicionais de monitoramento reagem a incidentes após limiares estáticos serem ultrapassados (ex: `CPU > 90%`). Este projeto implementa uma abordagem **AIOps (Artificial Intelligence for IT Operations)** na borda (edge). O agente coleta métricas localmente e utiliza um modelo de Machine Learning (via ONNX Runtime) para identificar padrões anômalos em séries temporais antes que resultem em falhas sistêmicas.

### 🎯 Objetivos Principais
- **Baixo Overhead:** O agente não deve consumir os recursos que está protegendo.
- **Inferência Local:** Execução de modelos de ML em Go sem depender de chamadas de rede ou ambientes Python pesados.
- **Predição:** Alertar sobre comportamentos atípicos usando algoritmos clássicos de detecção de anomalias (ex: Isolation Forest).

---

## 🏗️ Arquitetura do Sistema

O fluxo de dados ocorre inteiramente em memória através de *Goroutines* e *Channels* para garantir alta concorrência e zero bloqueios de I/O.

1. **Sensor (Coletor):** Lê dados do `/proc` ou via `gopsutil` (CPU, RAM, Disco, Rede).
2. **Buffer de Janela Deslizante (Sliding Window):** Mantém os últimos $N$ pontos de dados em memória para contexto histórico.
3. **Motor de Inferência (ONNX):** Avalia a janela temporal contra um modelo de *Machine Learning* pré-treinado.
4. **Atuador (Alertas):** Emite um *Anomaly Score* e dispara webhooks caso o limiar de segurança seja rompido.

---

## 🚀 Roadmap de Desenvolvimento

- [ ] **Fase 1: Ingestão de Dados**
  - Implementar coleta de métricas com `shirou/gopsutil`.
  - Configurar concorrência com Goroutines.
- [ ] **Fase 2: Estruturas em Memória**
  - Desenvolver o buffer de janela deslizante (Ring Buffer).
  - Implementar normalização de dados (Z-score/Min-Max) nativa em Go.
- [ ] **Fase 3: Inteligência e Inferência**
  - Treinar modelo Isolation Forest/Autoencoder off-line (Python/Scikit-learn).
  - Exportar modelo para `.onnx`.
  - Integrar runtime do ONNX ao Go para inferência das métricas do buffer.
- [ ] **Fase 4: Alertas e CLI**
  - Interface de linha de comando para visualização local.
  - Integração de disparos via Webhook (Slack/Discord).

---

## 🛠️ Stack Tecnológica

* **Linguagem:** [Go (Golang)](https://go.dev/)
* **Coleta de Métricas:** [gopsutil](https://github.com/shirou/gopsutil)
* **Runtime de IA:** [ONNX Runtime](https://onnxruntime.ai/) (Binding para Go)
* **Testes de Stress (Validação):** `stress-ng` (Linux)

---

## 💻 Como Executar (Ambiente de Desenvolvimento)

*Nota: As instruções abaixo refletem a arquitetura planejada.*

1. Clone o repositório:
```bash
git clone [https://github.com/seu-usuario/aiops-node-agent.git](https://github.com/seu-usuario/aiops-node-agent.git)
cd aiops-node-agent
