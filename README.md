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



------------------------------------------------------
Criar o modelo 100% em Go é uma excelente demonstração de fundamentos em ciência da computação. Isso elimina o ecossistema do Python, remove dependências de C (CGO/ONNX) e resulta em um binário puramente estático, o que é o "santo graal" para agentes de infraestrutura.

Como você construirá a inteligência artificial do zero, sem bibliotecas externas, a Isolation Forest é o algoritmo ideal. Ele baseia-se em estruturas de dados (árvores binárias) em vez de cálculos matriciais complexos, casando perfeitamente com a tipagem forte do Go.

Aqui está a arquitetura para implementar isso usando apenas a biblioteca padrão (como math e math/rand):

1. Modelagem das Estruturas de Dados
Você representará as florestas e árvores usando structs e ponteiros.

Go
type Point []float64 // Representa [CPU, RAM, Disco]

type Node struct {
	Left        *Node
	Right       *Node
	SplitColumn int     // Qual métrica foi usada para dividir
	SplitValue  float64 // O valor do limiar da divisão
	Size        int     // Quantidade de pontos neste nó
}

type IsolationForest struct {
	Trees       []*Node
	SampleSize  int
	MaxTreeSize int
}
2. A Fase de "Treinamento" (Construção da Árvore)
O treinamento consiste em gerar árvores aleatórias até isolar cada amostra de métrica coletada no período de warm-up do seu servidor.

A Lógica: Crie uma função recursiva buildTree(data []Point, currentDepth int) *Node.

Em cada nó, escolha aleatoriamente uma coluna (ex: Uso de CPU) e um valor aleatório entre o min e o max dessa métrica.

Divida os dados em dois slices (menores que o valor vão para a esquerda, maiores vão para a direita).

Pare a recursão quando o slice de dados tiver apenas 1 elemento ou a profundidade máxima for atingida.

3. A Fase de Inferência (Cálculo da Anomalia)
É aqui que o agente avalia as métricas em tempo real (a janela deslizante).

Crie uma função pathLength(x Point, n *Node, currentDepth int) float64.

Ela percorre a árvore descendo para a esquerda ou direita dependendo do valor da métrica de x.

O retorno é a profundidade em que o ponto foi isolado.

O Segredo Matemático: Faça a média do comprimento desse caminho em todas as árvores da sua estrutura IsolationForest. Pontos anômalos (como um pico bizarro de CPU) serão isolados muito rápido, resultando em um caminho curto.

4. Gestão de Memória e Concorrência (Apontamento Técnico)
Se o agente roda continuamente, a floresta precisa ser atualizada para não gerar falsos positivos com mudanças de padrão normais (concept drift).
Em vez de retreinar tudo, use um Ring Buffer para as árvores. Crie uma Goroutine que, a cada X horas, descarta a árvore mais antiga do slice Trees e treina uma nova com os dados recentes, garantindo uso constante de memória pelo Garbage Collector do Go.

Você prefere começar modelando a estrutura das árvores binárias ou focando primeiro no coletor bruto de métricas do sistema operacional para gerar o dataset inicial?
