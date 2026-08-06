# Radar de Vagas — formulário de cadastro

Página única, sem dependências, servida pelo GitHub Pages. É o que o cliente
abre para entrar no Radar de Vagas.

**Link:** `https://<usuario>.github.io/radar-de-vagas-intake/`

O motor que consome estas respostas fica em outro repositório, privado.

---

## Configurar

No topo do `<script>` em `index.html`:

```js
const CONFIG = {
  ENDPOINT: "",              // URL /exec do Apps Script da Central de Cadastros
  CHAVE:    "",              // a mesma senha do topo do intake-apps-script.gs
  WHATSAPP: "5511999999999", // só dígitos, formato internacional
  OPERADOR: "Ramon"
};
```

Com `ENDPOINT` vazio o formulário funciona igual e termina no download do JSON
— útil para testar antes de publicar o endpoint.

`CHAVE` fica visível no código-fonte desta página. Ela não é autenticação: serve
para barrar robô que varre URLs de Apps Script no escuro. O que protege de
verdade é o endpoint nunca devolver dados.

---

## Pré-preencher os cargos sugeridos

Depois de ler o currículo do cliente, você devolve um link com os três cargos já
propostos. O parâmetro `?s=` é um JSON em base64 url-safe:

```json
{
  "nome": "Maria Silva",
  "analise": "Uma frase sobre o que os dois últimos empregos dizem.",
  "cargos": [
    {"titulo": "Product Owner", "porque": "por que este cargo"},
    {"titulo": "Product Manager", "porque": "..."},
    {"titulo": "Product Operations", "porque": "..."}
  ]
}
```

O cliente aceita ou recusa cada um e acrescenta os dele. Recusar é informação
tão boa quanto aceitar.

---

## As dez etapas

| # | Etapa | O que decide no motor |
|---|---|---|
| 0 | Abertura | — |
| 1 | Quem é você | `situacao` define o corte de score e a cadência |
| 2 | Currículo e os 2 últimos empregos | Base da análise dos cargos; o cargo atual vira cargo adjacente |
| 3 | Cargos-alvo | As consultas enviadas aos portais |
| 4 | Senioridade | Bônus e penalidade por nível no título |
| 5 | Movimento de carreira | Destino × origem, e se o assunto é exigido |
| 6 | Modalidade e local | Abrangência **por modalidade** — remoto e presencial quase nunca têm o mesmo alcance |
| 7 | Empresas-alvo, segmentos e porte | Bônus por empresa; vocabulário de segmento alcança empresa que o cliente não conhece |
| 8 | Empresas a evitar e deal-breakers | Exclusão dura: palavra no título descarta a vaga |
| 9 | Regime, salário, inglês, especialidades, cadência | Especialidade marca a coluna filtrável do tracker |
| 10 | Consentimento e envio | LGPD |

---

## Detalhes de implementação que importam

- **Estado no `localStorage`.** São 10 minutos de formulário no celular; sem
  isso, uma ligação no meio custa o cadastro inteiro.
- **`Content-Type: text/plain` no POST.** Evita o preflight do CORS, que o Apps
  Script não responde.
- **Fallback em duas camadas.** Se o `fetch` normal falhar, repete em
  `no-cors` (o POST chega, a resposta é que não pode ser lida) e a tela final
  passa a pedir o download do JSON. Cadastro não se perde por rede ruim.
- **Nada de dependência externa.** Sem CDN, sem fonte remota, sem framework.
  Abre offline e continua abrindo daqui a cinco anos.

---

## Testar local

```bash
python3 -m http.server 8788 --directory .
```

E abra `http://localhost:8788/`. Para testar o pré-preenchimento, cole um `?s=`
gerado como acima — abrir o arquivo direto por `file://` funciona, mas a query
string se perde em alguns visualizadores.
