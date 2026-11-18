Projeto de Gêmeo Digital Veicular
Especificação técnica de arquitetura, coleta, processamento e sincronização com BeamNG.tech

Versão: 2.0
Simulador: BeamNG.tech
Integração: Python + BeamNGpy
Backend: Python + FastAPI
Comunicação: WebSocket
Objetivo: Reproduzir em tempo real o comportamento de um veículo real em uma réplica virtual da pista, utilizando um modelo físico ativo no BeamNG.tech.

1. Visão geral

O projeto consiste no desenvolvimento de um sistema capaz de coletar informações de um veículo real, processar esses dados e utilizá-los para controlar um veículo virtual no BeamNG.tech.

O veículo real será instrumentado por meio de:

Smartphone instalado no interior do veículo.
Adaptador OBD-II conectado à porta de diagnóstico.
Aplicativo responsável pela coleta e transmissão dos dados.

As informações serão transmitidas para um servidor desenvolvido em Python com FastAPI.

O backend será responsável por processar os dados, estimar o estado de movimento do veículo real, calcular as referências de controle e enviá-las ao cliente Python responsável pela integração com o BeamNG.tech.

O BeamNG.tech já possui um modelo do veículo e uma réplica da pista com dimensões e características próximas das reais.

O principal objetivo desta etapa é fazer com que o veículo virtual reproduza o comportamento do veículo real, mantendo a física própria do simulador ativa.

Não existe disponibilidade garantida do ângulo real do volante nem da velocidade individual das rodas. Portanto, o controle será desenvolvido utilizando os estados de movimento estimados a partir dos sensores disponíveis.

1.1 Objetivos
Expandir o aplicativo existente para coletar dados adicionais do smartphone e do OBD-II.
Implementar comunicação em tempo real entre aplicativo e FastAPI.
Desenvolver sincronização temporal dos dados.
Implementar processamento e fusão sensorial.
Estimar posição, orientação, velocidade, aceleração e trajetória.
Desenvolver controladores para reproduzir os estados observados.
Implementar um cliente Python utilizando BeamNGpy.
Integrar o cliente ao backend por comunicação bidirecional.
Obter continuamente o estado físico do veículo virtual.
Comparar os estados reais e virtuais e avaliar a fidelidade da reprodução.
1.2 Resultado esperado

Ao conduzir o veículo real em uma pista, o veículo virtual deverá reproduzir:

Posição espacial.
Orientação.
Velocidade.
Aceleração longitudinal.
Aceleração lateral.
Rotação em torno do eixo vertical (Yaw Rate).
Trajetória percorrida.
Comportamento de aceleração, frenagem e curvas.

A reprodução deverá ocorrer utilizando a física do BeamNG.tech, sem depender exclusivamente de alterações artificiais de posição e orientação.

2. Arquitetura geral

A arquitetura será dividida em cinco componentes:

Aplicativo de coleta.
Backend FastAPI.
Módulo de estimação e controle.
Cliente Python BeamNGpy.
Simulador BeamNG.tech.
2.1 Diagrama da arquitetura
mermaid
flowchart TD
    A[Veículo real] --> B[Smartphone]
    C[Adaptador OBD-II] --> B

    B --> D[Aplicativo de coleta]
    D --> E[Calibração e sincronização]
    E --> F[WebSocket]

    F --> G[Backend FastAPI]
    G --> H[Validação e armazenamento]
    H --> I[Fusão sensorial]
    I --> J[Estimador de estado]
    J --> K[Controlador veicular]

    K --> L[Comandos de controle]
    L <--> M[WebSocket BeamNG]

    M <--> N[Cliente Python BeamNGpy]
    N <--> O[BeamNG.tech]

    O --> P[Estado físico do veículo virtual]
    P --> N
    N --> G

2.2 Responsabilidades dos componentes
Aplicativo de coleta
Conectar ao adaptador OBD-II.
Coletar GPS.
Coletar acelerômetro.
Coletar giroscópio.
Coletar orientação do smartphone.
Padronizar unidades.
Registrar timestamps de aquisição.
Transmitir os dados via WebSocket.
Monitorar conexão e qualidade dos sensores.
Backend FastAPI
Receber os dados do aplicativo.
Validar os pacotes.
Organizar leituras temporalmente.
Sincronizar sensores.
Processar e filtrar informações.
Executar fusão sensorial.
Estimar o estado do veículo real.
Calcular referências e comandos de controle.
Receber telemetria do BeamNG.
Comparar estados reais e virtuais.
Registrar sessões e indicadores de desempenho.
Módulo de estimação e controle
Converter coordenadas geográficas para coordenadas locais.
Estimar orientação e trajetória.
Calcular curvatura.
Calcular referências de esterçamento.
Controlar velocidade.
Comparar o estado real com o virtual.
Corrigir erros de sincronização.
Cliente Python BeamNGpy
Conectar ao BeamNG.tech.
Inicializar cenário e veículo.
Receber comandos do backend.
Aplicar comandos ao veículo.
Controlar o avanço da simulação.
Coletar estado físico do veículo virtual.
Coletar sensores virtuais.
Transmitir telemetria ao backend.
Monitorar falhas de comunicação.
BeamNG.tech
Executar a simulação física.
Calcular a dinâmica do veículo.
Processar comandos de direção, acelerador e freio.
Atualizar os estados físicos.
Disponibilizar informações por meio do BeamNGpy.
3. Fase 0 — Evolução do aplicativo de coleta

O aplicativo existente será expandido para coletar os dados necessários ao projeto.

3.1 Dados OBD-II
Informação	PID	Unidade	Prioridade
Velocidade do veículo	0D	km/h	Essencial
RPM	0C	rpm	Alta
Posição do acelerador	11	%	Alta
Carga calculada do motor	04	%	Média
Temperatura do motor	05	°C	Baixa
Posição absoluta do acelerador	49, se suportado	%	Média

Os PIDs deverão ser consultados conforme a disponibilidade informada pela ECU.

A velocidade geral do veículo será suficiente para iniciar a implementação.

3.2 Dados GPS
Informação	Unidade
Latitude	Graus
Longitude	Graus
Altitude	m
Velocidade	m/s
Direção de deslocamento	Graus
Precisão horizontal	m
Precisão da velocidade	m/s
Timestamp	ms
3.3 Acelerômetro

Coletar os três eixos:

X.
Y.
Z.

Unidade: m/s².

Frequência desejável: 50–100 Hz.

Deverá ser realizada compensação da gravidade e transformação dos eixos para o referencial do veículo.

3.4 Giroscópio

Coletar:

Rotação X.
Rotação Y.
Rotação Z.

Unidade: rad/s.

Frequência desejável: 50–100 Hz.

O eixo Z transformado para o referencial do veículo será utilizado para estimar o Yaw Rate.

3.5 Orientação

Utilizar os recursos de orientação disponíveis no sistema operacional.

No Android:

TYPE_ROTATION_VECTOR.
TYPE_GAME_ROTATION_VECTOR.
TYPE_ACCELEROMETER.
TYPE_GYROSCOPE.

No iOS:

Core Motion.

A orientação deverá ser utilizada para transformar as medições da IMU para o referencial do veículo.

3.6 Calibração

O smartphone deverá permanecer fixo durante a coleta.

A calibração deverá identificar:

Orientação do aparelho.
Offset do giroscópio.
Orientação dos eixos.
Influência da gravidade.
Convenção de sinais.

Convenção recomendada:

X positivo para frente.
Y positivo para esquerda.
Z positivo para cima.
Yaw positivo para esquerda.
4. Fase 1 — Comunicação com o FastAPI

O aplicativo deverá manter uma conexão WebSocket persistente com o backend.

Endpoint:

text
ws://servidor:8000/ws/vehicle/{session_id}


Em produção, utilizar WSS.

4.1 Estrutura de telemetria

Cada leitura deverá possuir timestamp de aquisição próprio.

Exemplo:

json
{
  "session_id": "session_001",
  "vehicle_id": "vehicle_001",
  "sequence": 1250,
  "gps": {
    "timestamp": 1794950000125,
    "latitude": -23.55052,
    "longitude": -46.63331,
    "speed": 15.2,
    "heading": 120.5,
    "horizontal_accuracy": 3.0
  },
  "imu": {
    "timestamp": 1794950000120,
    "accelerometer": {
      "x": 0.45,
      "y": 1.20,
      "z": 9.81
    },
    "gyroscope": {
      "x": 0.01,
      "y": 0.02,
      "z": 0.18
    }
  },
  "obd": {
    "timestamp": 1794950000115,
    "speed": 15.0,
    "rpm": 3200,
    "throttle_position": 35
  }
}


Os valores são ilustrativos.

4.2 Requisitos
Reconexão automática.
Controle de sequência.
Identificação de pacotes perdidos.
Controle de latência.
Monitoramento de disponibilidade dos sensores.
Buffer local.
Identificação de sessão.
Registro de falhas.
5. Fase 2 — Processamento e fusão sensorial

O backend deverá processar os dados em etapas independentes.

5.1 Padronização

Todas as unidades serão convertidas para:

Velocidade: m/s.
Aceleração: m/s².
Rotação: rad/s.
Ângulo: rad.
Distância: m.
Tempo: s.

Conversão da velocidade:

vm/s=vkm/h3.6
5.2 Transformação de coordenadas

Transformar os dados da IMU para o referencial do veículo.

Compensar a gravidade no acelerômetro.

Converter latitude e longitude para coordenadas locais ENU.

5.3 Sincronização temporal

O backend deverá:

Organizar leituras pelos timestamps.
Identificar leituras atrasadas.
Interpolar quando apropriado.
Identificar dados ausentes.
Produzir estados com timestamp de referência.
5.4 Fusão sensorial

Implementar inicialmente um filtro complementar.

Posteriormente, implementar um EKF.

Estado sugerido:

X=[x	y	v	ψ	r	ax	ay]T

Onde:

x/y: posição.
v: velocidade.
ψ: orientação.
r: Yaw Rate.
ax: aceleração longitudinal.
ay: aceleração lateral.

O estimador deverá considerar a incerteza dos sensores.

6. Fase 3 — Estimativa de trajetória e direção
6.1 Orientação
ψt=ψt−1+rΔt

A orientação estimada deverá ser corrigida utilizando informações do GPS e da fusão sensorial.

6.2 Curvatura
κ=rv

A estimativa deverá ser limitada ou desconsiderada em velocidades muito baixas.

Também deverá ser comparada com a curvatura calculada a partir da trajetória GPS.

6.3 Ângulo de esterçamento

Utilizar inicialmente o Kinematic Bicycle Model:

δ=arctan⁡(Lκ)

Onde L representa o entre-eixos do veículo virtual.

Esse valor é uma estimativa geométrica do esterçamento necessário.

6.4 Construção da trajetória

Cada ponto deverá conter:

Timestamp.
X.
Y.
Orientação.
Velocidade.
Curvatura.
Aceleração longitudinal.
Aceleração lateral.

A trajetória será utilizada como referência pelo controlador.

7. Fase 4 — Controle do veículo virtual

O controle deverá utilizar uma arquitetura em malha fechada.

7.1 Comparação de estados

Estado real:

Posição.
Orientação.
Velocidade.
Aceleração.
Yaw Rate.
Curvatura.

Estado virtual:

Posição.
Orientação.
Velocidade.
Aceleração.
Yaw Rate.
Esterçamento aplicado.

Erros:

ex=xreal−xvirtual
ey=yreal−yvirtual
ev=vreal−vvirtual
eψ=ψreal−ψvirtual

Os erros espaciais deverão ser calculados no mesmo referencial.

7.2 Direção

Utilizar Pure Pursuit inicialmente.

κd=2sin⁡(α)Ld
δd=arctan⁡(Lκd)

O controlador deverá considerar a velocidade para ajustar a distância de antecipação.

7.3 Velocidade

Utilizar a velocidade OBD como referência.

Um controlador PID poderá calcular os comandos de aceleração e frenagem.

7.4 Dinâmica

Aceleração longitudinal, aceleração lateral e Yaw Rate serão utilizados como referências adicionais para avaliar e ajustar o comportamento.

O controlador não deverá simplesmente impor essas acelerações ao veículo virtual sem considerar sua física.

8. Fase 5 — Cliente Python BeamNGpy

Esta fase implementará a comunicação entre o FastAPI e o BeamNG.tech.

O cliente Python deverá ser independente do backend e poderá ser executado na mesma máquina do simulador.

8.1 Responsabilidades
Inicializar o BeamNG.tech.
Conectar utilizando BeamNGpy.
Carregar o cenário.
Posicionar o veículo.
Configurar os sensores virtuais.
Receber comandos do backend.
Aplicar controles.
Avançar a simulação.
Coletar estados físicos.
Enviar telemetria ao backend.
8.2 Estrutura sugerida
text
beamng_client/
│
├── main.py
├── beamng_connection.py
├── scenario_manager.py
├── vehicle_controller.py
├── telemetry_collector.py
├── websocket_client.py
├── simulation_loop.py
├── sensor_manager.py
└── config.py

8.3 Integração com BeamNGpy

A comunicação com o simulador deverá utilizar a API oficial BeamNGpy.

Principais recursos:

BeamNGpy: conexão com o simulador.
Scenario: gerenciamento do cenário.
Vehicle: gerenciamento do veículo.
Vehicle.control(): aplicação de comandos.
Vehicle.state: consulta do estado.
Sensores virtuais: coleta de informações complementares.

Sensores que podem ser avaliados:

Sensor	Utilidade
Electrics	Acelerador, freio e dados elétricos
GForces	Acelerações
Timer	Tempo da simulação
AdvancedIMU	Informações inerciais
Estado do veículo	Posição, velocidade e orientação

A disponibilidade e os campos específicos deverão ser confirmados na versão instalada do BeamNGpy.

Documentação:

https://beamngpy.readthedocs.io/
https://www.beamng.tech/
8.4 Ciclo de simulação

O cliente deverá executar um ciclo controlado.

mermaid
flowchart TD
    A[Iniciar ciclo] --> B[Obter comando válido]
    B --> C[Aplicar controles]
    C --> D[Avançar simulação]
    D --> E[Consultar estado]
    E --> F[Coletar sensores virtuais]
    F --> G[Enviar telemetria]
    G --> H{Simulação ativa?}
    H -->|Sim| B
    H -->|Não| I[Encerrar]

8.5 Controle do tempo

O avanço da simulação deverá ser controlado pelo cliente BeamNG.

Como referência inicial, utilizar:

Δt=0,01s

Isso representa um passo de 10 ms, equivalente a 100 passos por segundo.

O valor deverá ser validado conforme a capacidade de processamento e a configuração da simulação.

O ciclo deverá utilizar os mecanismos de avanço controlado de tempo disponibilizados pela versão do BeamNGpy instalada.

O envio e recebimento de dados pela rede não deverá determinar o avanço físico do simulador.

Caso o backend não envie um novo comando dentro do prazo, o cliente deverá aplicar uma política definida de último comando válido, limite de validade e comportamento seguro.

8.6 Organização dos processos

Recomenda-se executar:

Aplicativo no smartphone.
FastAPI como processo de servidor.
BeamNG.tech como processo de simulação.
Cliente BeamNGpy como processo independente.

Essa separação facilita testes, reinicializações e manutenção.

9. Fase 6 — Comunicação bidirecional FastAPI e BeamNGpy

O cliente BeamNGpy e o backend deverão manter uma conexão persistente.

Endpoint sugerido:

text
ws://servidor:8000/ws/simulator/{session_id}


O protocolo deverá suportar mensagens de controle, telemetria, sincronização e status.

9.1 Comandos enviados pelo backend

Exemplo:

json
{
  "type": "control_command",
  "session_id": "session_001",
  "sequence": 1250,
  "reference_timestamp": 1794950000125,
  "valid_until": 1794950000225,
  "simulation_dt": 0.01,
  "control": {
    "steering": 0.12,
    "throttle": 0.35,
    "brake": 0.0
  }
}


Os valores são ilustrativos.

Os controles deverão respeitar a convenção e os limites esperados pelo BeamNGpy.

O comando deverá possuir validade temporal para evitar aplicação de referências antigas.

9.2 Telemetria enviada pelo cliente BeamNG

Exemplo:

json
{
  "type": "virtual_telemetry",
  "session_id": "session_001",
  "sequence": 1250,
  "simulation_time": 12.50,
  "wall_time": 1794950000135,
  "vehicle_state": {
    "position": {
      "x": 125.2,
      "y": 82.0,
      "z": 0.4
    },
    "orientation": {
      "yaw": 1.23,
      "pitch": 0.01,
      "roll": 0.02
    },
    "velocity": {
      "x": 14.7,
      "y": 1.5,
      "z": 0.0
    },
    "acceleration": {
      "longitudinal": 0.42,
      "lateral": 1.15
    },
    "yaw_rate": 0.17
  },
  "controls_applied": {
    "steering": 0.12,
    "throttle": 0.35,
    "brake": 0.0
  }
}


Os campos deverão ser mapeados a partir das estruturas efetivamente retornadas pelo BeamNGpy.

9.3 Separação entre tempo simulado e tempo real

O sistema deverá manter dois relógios:

Tempo simulado:

Representa a evolução física dentro do BeamNG.tech.

Tempo real:

Representa o horário de aquisição e transmissão dos dados.

Essa separação será fundamental para calcular a latência e sincronizar as sessões.

O tempo simulado deverá ser monotônico dentro da sessão.

O backend deverá registrar a relação entre tempo real e tempo simulado.

9.4 Política de comunicação

O protocolo deverá contemplar:

Número sequencial das mensagens.
Timestamp de referência.
Timestamp de validade do comando.
Identificação da sessão.
Identificação do tipo de mensagem.
Confirmação de conexão.
Detecção de mensagens atrasadas.
Recuperação após desconexão.
Monitoramento de latência.

Não deverá ser permitido aplicar comandos antigos indiscriminadamente.

10. Fase 7 — Sincronização entre veículo real e virtual

O objetivo é manter os dois veículos sincronizados.

O backend será responsável por comparar continuamente os estados estimados.

10.1 Fluxo de sincronização
Receber telemetria real.
Atualizar o estimador de estado.
Obter o estado atual do BeamNG.
Comparar os estados.
Calcular erros.
Atualizar os controladores.
Enviar novos comandos.
Receber o estado virtual atualizado.
Repetir.
10.2 Estratégia de controle

A estratégia deverá combinar:

Pure Pursuit para acompanhamento de trajetória.
PID para velocidade.
Controle de orientação.
Correção de erros espaciais.
Monitoramento da dinâmica lateral.

A física do BeamNG deverá permanecer ativa.

O objetivo não será sobrescrever diretamente posição e orientação a cada atualização, mas reduzir os erros por meio de comandos físicos compatíveis com o veículo virtual.

10.3 Compensação de latência

A comunicação entre smartphone, servidor e simulador introduzirá atrasos.

Será necessário:

Medir a latência total.
Identificar a idade de cada referência.
Utilizar timestamps para alinhar os estados.
Prever o estado de referência quando necessário.
Limitar a aplicação de comandos atrasados.

A previsão deverá ser limitada ao horizonte em que o modelo de movimento permanecer confiável.

11. Fase 8 — Persistência e validação

Todas as sessões deverão ser armazenadas.

11.1 Dados armazenados
Telemetria original do OBD.
Telemetria original do GPS.
Dados originais da IMU.
Dados processados.
Estados estimados.
Trajetória real.
Comandos enviados ao BeamNG.
Estado físico retornado pelo BeamNG.
Tempo simulado.
Tempo real.
Latência.
Pacotes perdidos.
Erros de sincronização.

Os dados originais deverão ser preservados separadamente dos processados.

11.2 Indicadores
Indicador	Objetivo
Erro de posição	Diferença espacial
Erro de orientação	Diferença angular
Erro de velocidade	Diferença de velocidade
Erro de aceleração	Diferença dinâmica
Erro de Yaw Rate	Diferença de rotação
Latência	Atraso real-virtual
Pacotes perdidos	Qualidade de comunicação
Tempo de processamento	Desempenho do backend
Tempo de passo	Desempenho da simulação
11.3 Testes
Veículo parado.
Trajeto em linha reta.
Curvas abertas.
Curvas fechadas.
Aceleração.
Frenagem.
Sessão completa.

Cada etapa deverá validar a estimativa, o controle e a sincronização antes de avançar para a seguinte.

12. Estrutura final sugerida do projeto
text
digital_twin/
│
├── mobile_app/
│   ├── obd/
│   ├── gps/
│   ├── imu/
│   ├── calibration/
│   └── websocket/
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── models/
│   │   ├── ingestion/
│   │   ├── processing/
│   │   ├── trajectory/
│   │   ├── control/
│   │   ├── simulator/
│   │   └── storage/
│   └── tests/
│
├── beamng_client/
│   ├── main.py
│   ├── beamng_connection.py
│   ├── scenario_manager.py
│   ├── vehicle_controller.py
│   ├── telemetry_collector.py
│   ├── websocket_client.py
│   ├── simulation_loop.py
│   ├── sensor_manager.py
│   └── config.py
│
├── shared/
│   ├── schemas/
│   ├── protocols/
│   └── constants/
│
└── documentation/
    ├── architecture.md
    ├── data_dictionary.md
    └── integration.md

13. Cronograma lógico de desenvolvimento
Fase	Entrega
0	Aplicativo expandido
1	Comunicação do aplicativo com FastAPI
2	Processamento e fusão sensorial
3	Estimador de trajetória
4	Controladores de direção e velocidade
5	Cliente BeamNGpy
6	Comunicação bidirecional com BeamNG
7	Sincronização dos veículos
8	Validação e otimização

A integração com o BeamNG poderá começar em paralelo com o desenvolvimento do processamento dos sensores, utilizando comandos e trajetórias simulados para validar a comunicação.

14. Diretrizes finais

O sistema deverá manter uma separação clara entre aquisição, estimação, controle e simulação.

O aplicativo será responsável pela coleta.

O FastAPI será responsável pelo processamento, estimação e controle.

O cliente BeamNGpy será responsável pela execução dos comandos e coleta de telemetria virtual.

O BeamNG.tech será responsável pela simulação física.

A comunicação entre os componentes deverá utilizar timestamps, identificadores de sessão e números sequenciais.

A fidelidade do gêmeo digital dependerá principalmente de:

Qualidade dos dados coletados.
Sincronização temporal.
Precisão da estimativa de movimento.
Calibração física do veículo virtual.
Estabilidade dos controladores.
Baixa latência e previsibilidade da integração.

Diretriz principal: o veículo real deverá funcionar como referência de movimento, enquanto o BeamNG.tech deverá reproduzir esse comportamento por meio de sua própria simulação física, com o backend continuamente estimando e corrigindo as diferenças entre os dois veículos.