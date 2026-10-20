# Organização dos materiais do MPI NARD

## Estrutura recomendada

O repositório atual é um site estático em HTML/CSS. A estrutura abaixo é uma
proposta para uma futura migração para Next.js com App Router; ela não implica
que os arquivos do Drive já tenham sido importados ou que o projeto atual use
Next.js.

```text
public/
  banner/                     # Banners aprovados para divulgação
  imagens/
    povos/
      puri/                   # Apenas imagens autorizadas para publicação
      tupinikim/
      guarani/
  materiais-publicos/
    gerais/                   # PDFs, apresentações e comunicados públicos
    povos/
      puri/
      tupinikim/
      guarani/
      conselho-territorial/
    mapas/                    # Somente mapas aprovados para divulgação

src/
  app/
    page.tsx                  # Landing page
    povos/
      [povoId]/
        page.tsx              # Página pública de povo, comunidade ou instância
    api/                      # Endpoints protegidos, se necessários
  components/
    home/                     # Componentes da página inicial
    povos/                    # Componentes reutilizados nas páginas de povos
    layout/                   # Cabeçalho, navegação e rodapé
  data/
    povos.ts                  # Conteúdo público usado pelas rotas
  lib/
    auth.ts                   # Autorização para recursos privados, se aplicável

storage-privado/              # Fora de public/ e, preferencialmente, fora do Git
  documentos-restritos/
  dados-operacionais/
```

Não coloque conteúdo restrito em `public/`: tudo nessa pasta pode ser servido
diretamente ao navegador. Documentos privados devem ficar em storage fechado ou
ser entregues por uma API com autenticação e autorização. Não inclua dados
pessoais ou documentos restritos no repositório.

## Classificação dos itens informados

| Item | Categoria | Justificativa e condição |
|---|---|---|
| `/Puri` | **2 — Área dos povos** | Conteúdo específico de comunidades Puri; publicar apenas materiais autorizados. |
| `/CLPI Tupinikim e Guarani` | **2 — Área dos povos** | Conteúdo público de consulta e materiais regionalizados. Registros com dados pessoais ou circulação restrita ficam na categoria 3. Manter páginas próprias para cada povo; uma rota conjunta pode representar somente a atividade compartilhada. |
| `/Conselho Territorial` | **2 — Área dos povos** | Materiais públicos da instância de governança. Atas internas ou restritas ficam na categoria 3. |
| `/Estudos de Impactos e Danos (EIDs)` | **2 — Área dos povos** | Estudos e resultados aprovados para divulgação, associados aos territórios. Rascunhos e dados sensíveis ficam na categoria 3. |
| `/Documentos (Polifonica, FGV, ...)` | **3 — Backstage** | Tratar como restrito por padrão, especialmente se houver dados pessoais. Uma cópia expressamente aprovada para publicação pode ser classificada em 1 ou 2 conforme o conteúdo. |
| `/Pasta FUNAI` | **3 — Backstage** | Manter fora do frontend; usar apenas versões públicas e autorizadas, se houver. |
| `/Geoprocessamento` | **2 — Área dos povos** | Mapas e dados geográficos locais aprovados para publicação. Coordenadas ou informações territoriais sensíveis devem permanecer restritas na categoria 3. |
| `/Protocolo` | **2 — Área dos povos** | Protocolos públicos de consulta ou específicos de um povo. Versões internas ou restritas ficam na categoria 3. |
| `/Ata` | **3 — Backstage** | Atas de reuniões internas não devem ser expostas. Só publicar uma versão revisada e autorizada, se existir. |
| `/Informações gerais` | **1 — Landing page** | Resumos e informações gerais aprovados podem compor a apresentação institucional. Itens internos devem ser separados e protegidos. |
| `/Oficina Comunicação` | **1 — Landing page** | Materiais e comunicados de comunicação pública, após aprovação para divulgação. |
| `/Organograma _ Nova Equipe` | **3 — Backstage** | Organograma de equipe é material interno por padrão; publicar apenas uma versão institucional aprovada. |
| `/Trabalho técnico Capes` | **3 — Backstage** | Arquivos brutos de trabalho técnico ficam fora do cliente; resultados públicos resumidos podem ser publicados após revisão. |
| `CALENDÁRIO DE ATIVIDADES NARD.xlsx` | **3 — Backstage** | Manter a planilha operacional privada. Eventos públicos podem ser republicados separadamente, sem dados internos. |
| `CONCLUÍDAS` (logística, trabalhadores Puri etc.) | **3 — Backstage** | Controle de logística e trabalhadores pode conter dados pessoais e não deve ser publicado. |
| Banners, apresentações gerais e comunicados públicos | **1 — Landing page** | Publicar em `public/banner/` ou `public/materiais-publicos/gerais/` após autorização. |

As categorias 1 e 2 indicam destino para **cópias públicas aprovadas**, não
autorização automática para divulgar os arquivos do Drive. Em caso de dúvida
sobre consentimento, privacidade ou sensibilidade territorial, manter o
original restrito até validação.

## Exemplo de rota dinâmica (Next.js App Router)

Crie `src/app/povos/[povoId]/page.tsx`. O exemplo usa uma lista explícita de
slugs permitidos; assim, a URL não determina um caminho arbitrário de arquivo.
Ele pressupõe uma versão do Next.js em que `params` é uma `Promise`.

```tsx
import { notFound } from "next/navigation";

const povos = {
  puri: {
    nome: "Povos Puri",
    resumo: "Conteúdo público aprovado sobre as comunidades Puri.",
  },
  tupinikim: {
    nome: "Povos Tupinikim",
    resumo: "Conteúdo público aprovado sobre as comunidades Tupinikim.",
  },
  guarani: {
    nome: "Povos Guarani",
    resumo: "Conteúdo público aprovado sobre as comunidades Guarani.",
  },
  "tupinikim-guarani": {
    nome: "CLPI Tupinikim e Guarani",
    resumo: "Materiais públicos aprovados sobre atividades conjuntas de CLPI.",
  },
} as const;

type PovoId = keyof typeof povos;

function isPovoId(povoId: string): povoId is PovoId {
  return Object.hasOwn(povos, povoId);
}

export default async function PaginaPovo({
  params,
}: {
  params: Promise<{ povoId: string }>;
}) {
  const { povoId } = await params;

  if (!isPovoId(povoId)) {
    notFound();
  }

  const povo = povos[povoId];

  return (
    <main>
      <h1>{povo.nome}</h1>
      <p>{povo.resumo}</p>
    </main>
  );
}
```

As URLs individuais ficam em `/povos/puri`, `/povos/tupinikim` e
`/povos/guarani`; `/povos/tupinikim-guarani` representa apenas conteúdo
compartilhado de CLPI, sem substituir as páginas próprias de cada povo.
`notFound()` responde com a página 404 do Next.js para slugs desconhecidos.

Este exemplo é uma referência de migração: o site atual não tem dependências nem
configuração Next.js, portanto a rota não será executável até que o projeto seja
configurado para esse framework.
