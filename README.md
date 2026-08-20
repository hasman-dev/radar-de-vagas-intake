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
  EMAIL:    "voce@dominio.com", // contato de recuperação; NÃO use telefone pessoal
  OPERADOR: "Ramon"
};
```

Com `ENDPOINT` vazio o formulário funciona igual e termina no download do JSON
— útil para testar antes de publicar o endpoint.

**Nada de telefone pessoal aqui.** A página é pública e fica indexada; o
caminho de recuperação — para quando a Central não confirma o cadastro — usa o
e-mail do produto, não o seu número.

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
| 3 | Cargos-alvo | Pergunta primeiro **quais cargos a própria pessoa buscaria**, depois mostra a leitura do currículo. A lista dela vira consulta nos portais mas **não** pontua: se ela procura com o título errado — que é o caso que a pergunta existe para revelar — dar peso a isso ensinaria o motor a errar junto. A divergência aparece no `perfil.py --explicar`. |
| 4 | Senioridade | Bônus e penalidade por nível no título |
| 5 | Movimento de carreira | Destino × origem, e se o assunto é exigido |
| 6 | Modalidade e local | Abrangência **por modalidade** — remoto e presencial quase nunca têm o mesmo alcance |
| 7 | Empresas-alvo, segmentos e porte | Bônus por empresa; vocabulário de segmento alcança empresa que o cliente não conhece |
| 8 | Empresas a evitar e deal-breakers | Exclusão dura: palavra no título descarta a vaga |
| 9 | Regime, inglês, especialidades, cadência | Especialidade marca a coluna filtrável do tracker |
| 10 | Consentimento e envio | LGPD |

---

## O sistema visual

Direção vinda do Claude Design: **formulário como instrumento**. Vale conhecer
antes de mexer, porque cada peça carrega uma decisão.

| | |
|---|---|
| **Papel quente** | `#e9e7e2` na página, `#f5f3ee` no painel. Não é branco: um formulário de 10 minutos é leitura, e branco puro cansa. |
| **Régua vertical** | Os traços à esquerda são as 11 etapas. O ativo é mais longo e ganha o número. Clicável **só para trás** — pular adiante saltaria a validação. |
| **Duas vozes tipográficas** | Monoespaçada para o que é *medida* (etapa, rótulo, valor digitado). System-ui para o que é *fala* (pergunta e explicação). |
| **Campo como linha** | Entrada de texto é um fio, não uma caixa. Caixa dentro de painel é caixa dentro de caixa. |
| **Azul profundo** | `#2c527a`, 7,3:1 sobre o papel. Só o botão primário e os acentos o usam. |

**A régua continua vertical no celular.** Virar faixa horizontal a transformava
numa barra de progresso comum, e o elemento se perdia justamente onde a maioria
preenche. O trilho encolhe de 64px para 46px, e para 38px abaixo de 360px —
sobram 282px de conteúdo num iPhone SE, sem scroll horizontal. Só o número
some na largura menor; o traço ativo continua marcando a posição.

**Não use `.card` para agrupar.** O painel já é o cartão; bloco interno separa
por espaço e por um fio. Cartão dentro de cartão é o erro que o craft floor
chama de *nested cards are always wrong*.

**Rótulo em caixa-alta tem que ser curto.** Acima de ~20 caracteres a palavra
perde a forma que a torna legível — é por isso que "O que da sua experiência
sustenta essa transição?" virou "O que sustenta", com a pergunta inteira na
dica abaixo.

### Verificação

```bash
node .agents/skills/impeccable/scripts/detect.mjs --json index.html
```

Zero achados hoje. O detector precisa das dependências em `node_modules` para
rodar completo — sem elas ele subconta e parece limpo.

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
