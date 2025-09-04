# Sistema de Monitoramento por Colete (LoRa + Sensores)

## Escopo
Monitorar **localização** (GPS) e **batimentos cardíacos** (MAX30102), além de **movimento** (MPU-6050), enviados do **colete** via LoRa para **estações**. Os dados são armazenados em MySQL e associados a **treinos** e **atletas**, com **alertas** automáticos (ex.: taquicardia/queda).

## Requisitos da Entrega
- README explicando ideia/escopo e **relacionamentos entre entidades**.
- Modelo inicial no **MySQL Workbench** (`.mwb`).
- Repositório **público** em dupla (1 membro envia o link; o outro como colaborador).
- **Validação**: mínimo **5 entidades** e pelo menos **um** relacionamento de cada tipo (**1:1, 1:N, N:M**).

## Entidades & Relacionamentos (resumo)
- **Atleta** 1:1 **Colete (ativo)** via `AtletaColete`.
- **Colete** 1:N **Medicao** (cada leitura vem de um colete).
- **Estacao** 1:N **Medicao** (uma estação recebe muitas leituras).
- **Treino** N:M **Atleta** via `Participacao`.
- **Medicao** 1:1 **MedicaoGPS / MedicaoCardio / MedicaoInercial** (detalhes por tipo).
- **Alerta** 1:N **Medicao** (derivado de uma medição).

## Diagrama (EER)
O diagrama está no arquivo `modelo.mwb`. Principais tabelas:
`Atleta, Colete, Estacao, Treino, Participacao, Medicao, MedicaoGPS, MedicaoCardio, MedicaoInercial, Alerta, AtletaColete, SensorTipo`.

## Fluxo de Dados
1. Colete (SX1262 + sensores) coleta GPS, batimentos e IMU.
2. Pacote LoRa chega à **Estacao** e é processado/API → insere em `Medicao` e na tabela filha correspondente.
3. Regras geram **Alertas** quando limiares são excedidos.
4. Dados são vinculados a um **Treino** (opcional) e ao **Atleta** via colete ativo.

## Como reproduzir localmente
1. MySQL 8+.
2. Execute `sql/schema.sql` (fornecido) ou importe no Workbench (`Create EER Model from SQL Script`).
3. (Opcional) Use `samples/` com inserts de exemplo.

## Decisões de modelagem
- Tabelas filhas para separar payloads (GPS/Cardio/Inercial) → consultas mais limpas e índices específicos.
- `AtletaColete` mantém o **1:1** de “colete ativo”, permitindo histórico ao trocar o colete.
- Índices em (`id_colete`, `timestamp_coleta`) para consultas por tempo real.

## Próximos passos
- Endpoints de ingestão (REST/MQTT).
- Regras de alerta configuráveis por atleta.
- Dashboards (mapa + telemetria).
