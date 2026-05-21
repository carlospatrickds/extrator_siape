# 📘 Extrator de Fichas Financeiras SIAPE (Client-Side)

Aplicação web **100% local** para extração, filtragem e exportação de dados de fichas financeiras do SIAPE. Processa dois formatos de PDF diferentes diretamente no navegador, sem necessidade de servidor, Python ou instalação de dependências.

---

## ✨ Funcionalidades Principais
| Recurso | Descrição |
|--------|-----------|
|  **Privacidade Total** | Nenhum dado é enviado para servidores. Todo o processamento ocorre no navegador via JavaScript. |
| 📄 **Duas Abas Independentes** | `📘 Ficha Antiga` (layout com separadores `|`) e `📗 Ficha Nova` (layout livre com metadados). |
|  **Correção Automática de Texto** | Resolve caracteres corrompidos (`Ã‡ÃƒO`, ``) usando decodificação UTF-8 + reconstrução por contexto. |
| 🎛️ **Filtros Inteligentes** | Checkboxes com seleção padrão das 3 rubricas principais. Botões "Marcar Todas/Limpar" funcionais. |
| 📊 **Tabela em Tempo Real** | Atualiza instantaneamente ao marcar/desmarcar rubricas ou alterar o tipo (Receita/Desconto). |
| 📥 **Exportação Profissional** | CSV (Ficha Antiga) e Excel com 3 abas: Detalhado, Consolidado Anual e Relatório do Servidor (Ficha Nova). |
| 🔍 **Painel de Debug** | Permite visualizar o texto bruto extraído página a página para ajustes finos. |

---

## 🚀 Como Usar
1. Salve o arquivo como `index.html`
2. Abra em qualquer navegador moderno (Chrome, Edge, Firefox)
3. Escolha a aba correspondente ao formato do seu PDF
4. Arraste ou selecione o arquivo `.pdf`
5. Ajuste os filtros de rubricas e tipo
6. Clique em **Exportar CSV** ou **Exportar Excel**

---

##  Arquitetura Técnica
| Camada | Tecnologia |
|--------|------------|
| **Frontend** | HTML5 + CSS3 + Vanilla JS |
| **PDF Parsing** | `pdf.js` (Mozilla) via CDN |
| **Excel/CSV** | `SheetJS` (xlsx) via CDN |
| **Estado** | Máquina de estados simples (`BUSCA_INICIO → BUSCA_ANO → LEITURA`) |
| **Deploy** | Zero. Arquivo único, sem build, sem servidor. |

---

## 🚧 Maiores Dificuldades Enfrentadas & Soluções

| Desafio | Causa | Solução Implementada |
|---------|-------|----------------------|
| **Texto fragmentado no PDF** | PDFs não armazenam tabelas. `pdf.js` retorna itens com coordenadas X/Y isoladas. | Agrupamento por tolerância de `±3px` no eixo Y, ordenação por X e reconstrução de linhas lógicas. |
| **Codificação corrompida (`Ã‡ÃƒO`, ``)** | UTF-8 interpretado como Latin-1/Windows-1252 durante a geração do PDF original. | Conversão byte-a-byte + `TextDecoder('utf-8')` + mapa de substituição contextual quando falha. |
| **Losangos de substituição () persistentes** | Caractere Unicode `U+FFFD` usado pelo engine quando o glyph não existe no fonte do PDF. | Remoção programática + regras de reconstrução por prefixo/sufixo (`GRATIFICA` + `NATALINA` → `GRATIFICAÇÃO NATALINA AT`). |
| **Parser quebrava na página 1** | Comando `break` ao encontrar `TOTAL LÍQUIDO` interrompia o loop global. | Substituído por `continue` + reset de estado ao detectar novo cabeçalho `Siape - Sistema...`. |
| **Botões "Marcar Todas/Limpar" não funcionavam** | IDs trocados e falta de sincronização entre DOM e estado da tabela. | Refatoração com `toggleAll('chk1', state)` + listeners unificados + atualização em cascata. |
| **Layouts radicalmente diferentes** | Ficha Antiga usa `|` como delimitador; Ficha Nova usa padrão `Nome Rubrica Valores`. | Dois parsers independentes: um baseado em `split('|')`, outro em regex `/(.+?)\s+((?:\d{1,3}(?:\.\d{3})*,\d{2}\s*)+)/`. |
| **Memória & Performance em PDFs grandes** | Processamento síncrono travava a UI. | Uso de `async/await`, chunking implícito do `pdf.js`, e renderização sob demanda via filtros. |

---

## 📘 Guia para Projetos Semelhantes

Se você precisar criar um extrator de dados estruturados a partir de PDFs governamentais ou corporativos, siga este fluxo:

### 1. 📐 Análise do PDF Alvo
- Abra o PDF em um editor de texto ou use `pdftotext` para ver como o texto é armazenado.
- Identifique: separadores (`|`, tabs, espaços), quebras de página, cabeçalhos fixos e rodapés.
- Verifique a codificação real (UTF-8, Latin-1, Windows-1252).

### 2. 🛠️ Escolha da Stack
- **Client-side** (recomendado para privacidade): `pdf.js` + `SheetJS` + Vanilla JS.
- **Server-side** (para PDFs muito grandes ou OCR): Python (`pdfplumber`, `camelot`, `pytesseract`) ou Node.js (`pdf2json`).

### 3. 🔍 Extração Robusta de Texto
```js
// Padrão ouro para PDFs tabulares sem estrutura nativa
1. Agrupe itens por Y (tolerância 2-5px)
2. Ordene por X dentro de cada grupo
3. Una com espaço, normalize múltiplos espaços
4. Preserve número da página original
```

### 4. 🧹 Normalização & Codificação
- Sempre tente `new TextDecoder('utf-8').decode(bytes)` primeiro.
- Crie um fallback com dicionário de substituições para padrões conhecidos.
- Use regex contextual quando a correção automática falhar.

### 5. 🧠 Parser Estruturado
- Implemente uma **máquina de estados** simples:
  ```
  BUSCA_CABEÇALHO → DETECTA_ANO/MÊS → LEITURA_VALORES → RESET
  ```
- Nunca use `break` em loops de página; use `continue` ou reset de estado.
- Valide estrutura mínima antes de extrair (ex: `cols.length > idx + meses.length`).

### 6. 🎛️ UI/UX & Filtros
- Gere checkboxes dinamicamente a partir dos dados extraídos.
- Mantenha estado separado entre UI e dados brutos.
- Sincronize contadores, tabelas e botões com eventos unificados.

### 7. 📤 Exportação & Validação
- Use `\uFEFF` (BOM) em CSVs para compatibilidade com Excel BR.
- Em Excel, separe abas por uso: Detalhado, Consolidado, Metadados.
- Sempre valide valores monetários com regex antes de parsear.

### 8. 🧪 Testes & Manutenção
- Mantenha um diretório `/exemplos` com PDFs de teste.
- Logue as primeiras 30-50 linhas extraídas no console/debug.
- Adicione regras de contexto modularmente (array de objetos `{pre, suf, result}`).

---

## 📦 Dependências (CDN)
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
```
*Nenhuma instalação via npm/pip necessária.*

---

## ⚖️ Aviso Legal
Este projeto é uma ferramenta de automação local. Os dados processados permanecem exclusivamente no dispositivo do usuário. Recomendado para uso interno, auditoria e controle pessoal. Não substitui validação oficial em sistemas governamentais.

---

💡 *Dica final: PDFs são essencialmente arquivos de impressão, não de dados. Trate cada novo modelo como um projeto de engenharia reversa, não como parsing de CSV.*
