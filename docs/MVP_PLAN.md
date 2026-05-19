# ClickBus — Plano de Implementação MVP

**Objetivo:** Entregar um totem de autoatendimento no Portão de Embarque com foco em acessibilidade e recuperação de passagem sem bilhete físico.

---

## Escopo do MVP

| # | Feature | Prioridade |
|---|---------|-----------|
| 1 | Acessibilidade (interface, leitor de tela, contraste, Libras) | ALTA |
| 2 | Recuperar passagem por código de reserva / CPF | ALTA |
| 3 | Totem de autoatendimento com leitura de QR Code | ALTA |

---

## Arquitetura do MVP

```
[App ClickBus] ──QR Code──► [Totem]
                                │
[Código de reserva / CPF] ──►  │──► API ClickBus ──► Valida passagem
                                │
                                └──► Impressão do bilhete (se necessário)
```

**Stack sugerida:**
- **Totem:** Electron (kiosk mode) ou React Native Web — roda no hardware do totem
- **API:** REST existente da ClickBus (endpoint de validação + busca por reserva)
- **QR Code:** biblioteca `react-qr-reader` ou equivalente
- **Integração App:** deep link `clickbus://checkin?token=<token>` + QR Code gerado no app

---

## Fases de Entrega

### Fase 1 — Fundação Acessível (Sprint 1–2)

Tudo é construído acessível desde o início — não retrofitado depois.

**Entregáveis:**
- [ ] Design system do totem: botões grandes (min. 48x48dp), alto contraste, fonte ≥18pt
- [ ] Componente de input com suporte a leitor de tela (ARIA labels)
- [ ] Modo alto contraste (toggle na tela inicial)
- [ ] Suporte a libras: vídeo contextual por etapa (arquivo .mp4 embutido, funciona offline)
- [ ] Navegação por teclado físico (acessibilidade motora)
- [ ] Tela inicial com 2 fluxos claros: **"Tenho QR Code"** / **"Não tenho o bilhete"**

**Critério de aceite:** Usuário com deficiência visual consegue completar o fluxo só com áudio.

---

### Fase 2 — Recuperação de Passagem (Sprint 3)

O passageiro que perdeu o bilhete ou não tem o app consegue embarcar.

**Entregáveis:**
- [ ] Tela: input de **código de reserva** (ex: `CB-12345`) ou **CPF**
- [ ] Integração com endpoint `GET /reservations/{code}` da API ClickBus
- [ ] Exibição dos dados da viagem (destino, horário, poltrona, nome)
- [ ] Botão: **Imprimir bilhete** (thermal print via driver do totem)
- [ ] Botão: **Liberar embarque** (marca como validado na API)
- [ ] Estado de erro com mensagem clara: reserva não encontrada, viagem já realizada, etc.
- [ ] Envio de código por SMS/e-mail caso o passageiro não lembre (se API suportar)

**Critério de aceite:** Passageiro sem app e sem bilhete físico embarca em < 2 min.

---

### Fase 3 — Leitura de QR Code + Integração App (Sprint 4)

Fluxo principal para quem já tem o app ClickBus.

**Entregáveis:**
- [ ] Câmera do totem ativa na tela inicial com área de leitura destacada
- [ ] Decodificação do QR Code gerado pelo app (`react-qr-reader` ou `zxing`)
- [ ] Validação do token QR junto à API (`POST /checkin/validate`)
- [ ] Tela de confirmação: nome, destino, poltrona — com opção de imprimir ou só embarcar
- [ ] Timeout de 30s na câmera → fallback para input manual (Fase 2)
- [ ] Log de cada transação para auditoria offline

**Critério de aceite:** Leitura do QR Code em < 3 segundos, validação < 5 segundos.

---

## Telas do Totem (MVP)

```
┌─────────────────────────┐
│  🔊  [Alto Contraste]   │  ← acessibilidade sempre visível
│                         │
│  Bem-vindo à ClickBus   │
│                         │
│  ┌─────────┐ ┌────────┐ │
│  │ QR Code │ │Reserva/│ │
│  │  (App)  │ │  CPF   │ │
│  └─────────┘ └────────┘ │
│                         │
│  [🤟 Libras]  [PT|EN|ES]│
└─────────────────────────┘
```

---

## Integrações Necessárias

| Sistema | Endpoint | Método | MVP? |
|---------|----------|--------|------|
| Validar QR Code (app) | `/checkin/validate` | POST | Sim |
| Buscar por código de reserva | `/reservations/{code}` | GET | Sim |
| Buscar por CPF | `/reservations?cpf={cpf}` | GET | Sim |
| Marcar embarque realizado | `/checkin/confirm` | POST | Sim |
| Reenviar código por SMS/e-mail | `/reservations/{id}/notify` | POST | Opcional |
| Impressora térmica | Driver local do OS | SDK | Sim |

> Se os endpoints não existirem na API atual, mockear com JSON local para o MVP e alinhar com o time de backend da ClickBus.

---

## Critérios de Acessibilidade (WCAG 2.1 AA)

- Contraste mínimo 4.5:1 (texto normal) e 3:1 (texto grande)
- Toda ação possível via teclado físico
- Foco visível em todos os elementos interativos
- Mensagens de erro lidas por screen reader
- Vídeos de Libras em cada etapa crítica
- Botão de emergência físico no totem → chama agente humano
- Sessão encerrada automaticamente após 2 min de inatividade (privacidade)

---

## Fora do Escopo (MVP)

- Digital Twin / navegação 3D
- Realidade aumentada
- Câmeras de monitoramento de fila
- Múltiplos idiomas além de PT-BR (pode ser adicionado na Fase 1 sem custo extra)
- Ajuste de altura do totem (hardware — documentar requisito para fornecedor)

---

## Timeline

| Semana | Entrega |
|--------|---------|
| 1–2 | Design system acessível + tela inicial |
| 3 | Fluxo recuperação por código/CPF + integração API |
| 4 | Leitura QR Code + integração App |
| 5 | Testes de acessibilidade com usuários reais + ajustes |
| 6 | Deploy piloto (1 terminal) |

---

## Riscos

| Risco | Mitigação |
|-------|-----------|
| API ClickBus não tem endpoints necessários | Mock local + spec dos endpoints entregue ao backend na Sprint 1 |
| Hardware do totem sem suporte a câmera | Fallback para input manual já está no escopo |
| Conectividade instável no terminal | Cache offline da última sessão; fila de confirmações |
| Validação de QR Code com latência alta | Timeout de 5s + fallback para código manual |
