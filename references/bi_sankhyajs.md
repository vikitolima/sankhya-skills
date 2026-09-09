# BI-SankhyaJS para componentes de BI

## Origem e alcance

Fonte comunitária: [wansleynery/BI-SankhyaJS](https://github.com/wansleynery/BI-SankhyaJS).
Inspeção em 2026-09-09, commit `69d917594a2b781c7894976ba61f2fdf2d7fa515`.
As observações abaixo vêm da leitura do código; não houve execução em ambiente Sankhya.
O repositório disponibiliza componentes SankhyaJS/AngularJS em JSPs de BI. Não contém
um catálogo de gráficos nem comprova compatibilidade com todas as versões do ERP.

Arquivos consultados, fixados na revisão analisada:

- [README](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/readme.md): importação e ressalvas do autor.
- [JSP entidade](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/paginas/entidade/index.jsp) e [controladora](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/paginas/entidade/ControladoraCentral.component.js).
- [JSP standalone](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/paginas/standalone/index.jsp) e [controladora](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/paginas/standalone/ControladoraCentral.component.js).
- [Serviço de impressão](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/servicos/Impressao.js).
- [Licença MIT](https://github.com/wansleynery/BI-SankhyaJS/blob/69d917594a2b781c7894976ba61f2fdf2d7fa515/LICENSE), copyright 2023 W. N. Soto. Ao copiar código ou partes substanciais, incluir o aviso e a licença original no pacote. Esta referência contém análise, não uma cópia dos templates.

## Escolher a base

| Necessidade | Base | O que verificar |
|---|---|---|
| Indicadores, gráficos e consultas em gadget existente | Fluxo JSP nativo da skill | Preservar a arquitetura existente quando ela atende à solicitação |
| Grade/formulário ligado a entidade cadastrada | `paginas/entidade/index.jsp` | Nome de entidade reconhecido pelo Sankhya, metadados e permissões |
| Grade com resultado de consulta ou dados calculados | `paginas/standalone/index.jsp` | Campos declarados no dataset e correspondência com os objetos retornados |

O modelo entidade usa `sk-application` com `sk-dynaform`. A controladora define
`nomeEntidade`; `aoCarregarDynaform(dynaform, dataset)` guarda as instâncias, chama
`initAndRefresh()` e alterna para a grade com `goToGridView()`. Uma tabela física ou
view existir no Oracle não comprova que haja uma entidade utilizável com aquele nome.
Consultar metadados do ambiente; para tabelas e campos, carregar `sankhya-dicionario`.

O standalone usa `sk-dataset sk-standalone`, `sk-dsfield-md` e `sk-datagrid`, além de
`sk-double-face-panel`/`sk-form` para alternar as faces. Os callbacks observados são:

- `sk-dataset-created` → `aoCriarDataset(dataset)` guarda o dataset e o inicializa.
- `sk-refresh-handler` → `manipularAtualizar()` retorna uma Promise com um array de objetos.
- `sk-on-datagrid-loaded` → `aoCarregarDatagrid(datagrid)` guarda a grade.
- `sk-remove-handler` → `manipularRemover()` tenta mostrar uma mensagem; não implementa exclusão.

## Bootstrap e dependências

O JSP carrega recursos internos em `/mge/`, incluindo AngularJS, `snk.js` e `launcher.js`.
Depois importa controladoras e serviços usando `${BASE_FOLDER}`. A função
`startApplication()` registra o módulo e a controladora e executa `angular.bootstrap`.
Preservar a ordem de carga ao adaptar; não adicionar `ng-app` ou um segundo bootstrap.
Não mesclar automaticamente esse cabeçalho com o JSP nativo de `snk:load`.

O código também carrega Angular Material, SankhyaJX, SweetAlert2 e toastr por CDN.
JX e toastr usam URLs sem revisão fixa. A árvore do repositório não contém essas
bibliotecas nem os recursos internos do ERP: o ZIP, sozinho, não é autossuficiente.
Na implantação, conferir disponibilidade dos caminhos e fixar versões externas
testadas quando possível. Não inventar assinaturas de JX a partir de `executeQuery`:
o exemplo só demonstra `JX.consultar(sql)` retornando dados assíncronos.

Há valores de perfil/configuração embutidos (`PROFILEID`, `MGE_PARAMS`, entre outros)
e uma chamada com chave de AG Grid Enterprise. Eles precisam ser revisados contra o
ambiente de destino; a licença MIT deste repositório não concede direitos adicionais
sobre bibliotecas de terceiros. Não tratar esses valores como configuração universal.

## Adaptações recomendadas para dashboard de consulta

Estas são recomendações locais, não correções já homologadas no ERP:

1. Preservar a moldura do BI inicialmente. As duas controladoras chamam
   `JX.removerFrame`; segundo o README isso retira parâmetros laterais, recarga e
   maximização. O nome `TELA_HTML5` é necessário para a configuração original dessa
   chamada, não uma exigência universal para todo dashboard. Se a remoção for desejada,
   ajustar `instancia` ao nome exato e `paginaInicial` ao caminho relativo, sem `BASE_FOLDER`.
2. Configurar a interface para consulta. O standalone exibe botões de CRUD, embora
   não implemente o fluxo completo. Exibir esses botões como se houvesse persistência
   é um exemplo inadequado; ocultar adicionar/editar/salvar/remover em uma tela de
   consulta é a adaptação apropriada. Ocultar botões não substitui permissões no servidor.
3. Corrigir as referências de escopo ao adaptar o standalone: o painel usa
   `sk-dataset="$ctrl.objetoDS"`, enquanto a controladora e os demais componentes usam
   `central.dataset`. Para manter esse alias, usar `sk-dataset="central.dataset"` e
   validar a alternância grade/formulário no ERP.
4. O handler de remoção referencia `this.dicionario`, que não é inicializado na
   controladora analisada. Não reutilizar esse handler como exclusão pronta; em uma
   tela de consulta, remover a ação e seu vínculo.
5. No JSP entidade, a diretiva de arraste usa `$timeout` sem injetá-lo e captura o
   erro sem tratamento. Caso o recurso seja necessário, corrigir a injeção e tornar
   a falha visível; caso contrário, dispensar a diretiva na adaptação.
6. Substituir o `SELECT *` demonstrativo por colunas explícitas, filtros e limite ou
   paginação apropriados ao volume. Confirmar estruturas no dicionário e objetos
   `AD_` no projeto. Não concatenar entrada livre em SQL; consultar a API efetivamente
   disponível para parametrização. Prever carregamento, resultado vazio e erro.
7. Tratar `servicos/Impressao.js` como exemplo específico: possui parâmetros de
   relatório fixos e SQL com interpolação. Só incorporá-lo se impressão fizer parte
   do pedido, após revisar contrato, sessão e parâmetros do relatório no ambiente.

## Empacotamento e homologação

O README orienta importar o ZIP no **Construtor de Componentes do BI**, em componente
do tipo **HTML5**, escolhendo um dos JSPs como página inicial. Preservar `paginas/`
e `servicos/` na raiz do pacote, sem adicionar uma pasta externa de repositório.
O ponto de entrada é `paginas/entidade/index.jsp` ou `paginas/standalone/index.jsp`.
Não achatar essas subpastas: os imports usam seus caminhos. A árvore analisada não
fornece `tdb_dashboard.xml`; a importação do componente é distinta do fluxo de gadget
XML descrito na skill principal.

Antes de considerar uma adaptação pronta, registrar a versão do Sankhya e validar:

- JSP compilado, imports sem 404 e bootstrap único, sem erros no console.
- Grade e formulário vinculados ao mesmo dataset; carregamento e recarga funcionais.
- Resultado vazio, falha de consulta e volume representativo tratados.
- Parâmetros, recarga e maximização preservados quando a moldura for mantida.
- Usuário com permissões limitadas consegue apenas as operações autorizadas.

A leitura do código confirma padrões de implementação; a compatibilidade operacional
permanece pendente até esses testes ocorrerem no ambiente de destino.
