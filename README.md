# Pipeline somatico - DO VCF Anotado ao CGI (Câncer Genome Interpreter)

# 1 - Como obter o Token no CGI:
# https://www.cancergenomeinterpreter.org/rest_api

<img width="1919" height="427" alt="image" src="https://github.com/user-attachments/assets/5eed1fda-b2fe-4bfd-b553-814214d71f0b" />
Desce até o final da pagina e pega o Token (Não esquece de fazer o Login)

## Passo via collab: Caso queira fazer via Google Collab, só baixar esse arquivo aqui. Ele ta pronto pra uso, pode confiar :) (é um print, não tenta baixar pela imagem)
<img width="395" height="46" alt="image" src="https://github.com/user-attachments/assets/3573f900-78c8-49b9-941a-0d430f948e4a" />


## Passo 1: Clonar o GitHub do LMA Brasil, seguindo o comando para que ele funcione
```bash
git clone https://github.com/renatopuga/lmabrasil-hg38.git
```
## Passo 2: Usar o comando para filtrar o arquivo, deixando ele aceitavel para o GCI
```bash
OUTPUT="df_WPALL-cgi.txt"


FIRST_FILE=$(ls /content/lmabrasil-hg38/vep_output/liftOver_WP*_hg19ToHg38.vep.filter.tsv | head -n 1)


cut -f1-4 "$FIRST_FILE" | head -n 1 | sed -e "s/CHROM/CHR/g" | awk '{print $0"\tSAMPLE"}' > "$OUTPUT"

# 2. PROCESSAR E JUNTAR OS DADOS

for FILE in /content/lmabrasil-hg38/vep_output/liftOver_WP*_hg19ToHg38.vep.filter.tsv; do

    # Extrai o ID da amostra (ex: WP017) do nome do arquivo usando grep
    SAMPLE_ID=$(basename "$FILE" | grep -o "WP[0-9]\+")

    cut -f1-4 "$FILE" | tail -n +2 | awk -v id="$SAMPLE_ID" -v OFS="\t" '{print $0, id}' >> "$OUTPUT"

done

# 3. VERIFICAÇÃO
echo "Arquivo gerado: $OUTPUT"
echo "Primeiras 5 linhas:"
head -n 5 "$OUTPUT"
echo "----------------"
echo "Contagem de linhas por amostra no arquivo final:"
# Isso mostra quantas linhas de cada WP ficaram no arquivo final
cut -f5 "$OUTPUT" | sort | uniq -c
```
### Output esperado:
<img width="431" height="141" alt="image" src="https://github.com/user-attachments/assets/a5c960e3-e593-4a1d-8021-785459471df7" />


- Resultado esperado: fonte Google Collab

## Passo 3: Enviando um request "requisição" a API do CGI
Esse código em python permite realizar a requisição a API via código PYTHON, também tem a opção via bash, mas para fins didaticos, sera explicado via python
- Obs: No lugar de Seu_TOKEN, colocar o Token obtido no CGI
```python
import requests
# cabeçalho 
headers = {'Authorization': 'kaneka4850@gmail.com Seu_TOKEN'}
payload = {'cancer_type': 'HEMATO', 'title': 'Somatic ALL', 'reference': 'hg38'}
# requisição
r = requests.post('https://www.cancergenomeinterpreter.org/api/v1',
                headers=headers,
                files={
                        'mutations': open('/content/df_WPALL-cgi.txt', 'rb')
                        },
                data=payload)
r.json() # Formato json para verificação
```
<img width="230" height="32" alt="image" src="https://github.com/user-attachments/assets/fc49cdb6-e14a-45a2-a964-31e1c4774f5a" />
Resultado esperado via google collab

## Passo 4: Verificação do Status do Job_ID, lembrando que o job_id é individual
- Obs: Coloca o Token e seu Job_id aqui, não copia sem mudar não! :)
```Python
import requests
job_id = input("Digite seu job_id (Obtido na 3º célula)")

headers = {'Authorization': 'kaneka4850@gmail.com Seu_TOKEN'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json()
```
<img width="586" height="162" alt="image" src="https://github.com/user-attachments/assets/a984b422-d20e-4656-9e65-c6fff91b6989" />
Resultado esperado via google collab

## Passo 5: Verificação das logs
- Obs: Não esquece de arrumar o Token e o Job_id viu?
```Python
import requests
job_id = input("Digite seu job_id (Obtido na 3º célula)")

headers = {'Authorization': 'kaneka4850@gmail.com Seu_TOKEN'}
payload={'action':'logs'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload)
r.json()
```
<img width="901" height="257" alt="image" src="https://github.com/user-attachments/assets/3a10b7ef-5059-4d91-ab4a-15473ca9468a" />
Resultado esperado via Google Collab


## Passo 6: Fazendo o Download dos arquivos

- Obs: Você ta mudando o Token e o Job_Id né?
```Python
import requests
job_id = input("Digite seu job_id (Obtido na 3º célula)")

headers = {'Authorization': 'kaneka4850@gmail.com Seu_Token'} # permissões do CGI
payload={'action':'download'} # passando o que é pra ele fazer
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers, params=payload) # requisições
with open('samplefinal.zip', 'wb') as fd:
    fd.write(r._content)
```
- Esse não vai gerar output, então não fica triste, não dando erro é o que importa
## Passo 7: 7. Descompactar o .zip utilizando unzip.

```Bash
unzip -o samplefinal.zip # Esse -o ignora se ja tiver alguma amostra feita antes
```


<img width="339" height="98" alt="image" src="https://github.com/user-attachments/assets/fa186ee0-4347-408f-ba37-bc728bb5270a" />



Resultado esperado:

<img width="280" height="206" alt="image" src="https://github.com/user-attachments/assets/bc7b8e66-8ff4-494d-946c-db956e0deef1" />

- input01.tsv
Arquivo de entrada da análise. Contém as variantes somáticas fornecidas (gene, posição, alteração, tipo), geralmente derivadas de um VCF.

- alterations.tsv
Resultado da interpretação das variantes. Lista quais mutações foram detectadas, se são drivers, tipo de alteração e anotações funcionais/clínicas.

- biomarkers.tsv
Tabela de biomarcadores clínicos associados às variantes. Relaciona mutações a terapias, níveis de evidência e tipo de câncer (A, B, etc.).

- summary.txt
Resumo geral da análise. Traz metadados (tipo de câncer, genoma, versão do CGI) e contagens globais de variantes, drivers e biomarcadores.


## Passo 8: Verificando o resultado das alterações: arquivo alterations.tsv
```Python
import pandas as pd
pd.read_csv('/content/alterations.tsv',sep='\t',index_col=False, engine= 'python')
```
### Tabela de Variantes (CGI Output)

| SAMPLE | CHR | POSITION | REF | ALT | CGI Prediction | Consequence | HGVSp (Proteína) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **WP017** | chr1 | 114716127 | C | T | **driver (boostDM)** | missense | p.Gly12Ser (G12S) |
| **WP017** | chr1 | 152304661 | G | C | passenger | missense | p.Arg3409Gly |
| **WP017** | chr11 | 115209621 | G | A | passenger | missense | p.Thr344Ile |
| **WP017** | chr12 | 132140028 | C | T | non-protein affecting | intron | p(?) |
| **WP017** | chr14 | 24119817 | G | A | passenger | missense | p.Arg338His |
| **WP017** | chr14 | 59727407 | G | A | passenger | missense | p.Ala426Val |
| **WP017** | chr15 | 24675943 | G | A | passenger | missense | p.Ala26Thr |
| **WP017** | chr16 | 67616834 | G | A | **driver (oncodriveMUT)** | missense | p.Glu348Lys |
| **WP017** | chr16 | 71389851 | G | A | passenger | missense | p.Glu268Lys |
| **WP017** | chr16 | 74452195 | A | C | non-protein affecting | intron | p(?) |
| **WP017** | chr19 | 35673516 | A | C | passenger | missense | p.Thr147Pro |
| **WP017** | chr19 | 45319445 | T | G | non-protein affecting | intron | p(?) |
| **WP017** | chr2 | 137450992 | G | A | passenger | missense | p.Arg1036Gln |
| **WP017** | chr2 | 218880905 | A | C | non-protein affecting | 5_prime_UTR | p(?) |
| **WP017** | chr21 | 41901750 | C | T | passenger | splice_acceptor | p(?) |
| **WP017** | chr21 | 43094667 | T | G | **driver (boostDM)** | missense | p.Gln157Pro |
| **WP017** | chr3 | 133380804 | G | A | passenger | missense | p.Asp365Asn |
| **WP017** | chr7 | 584561 | G | A | passenger | missense | p.Thr239Met |
| **WP019** | chr1 | 16031913 | G | A | non-protein affecting | intron | p(?) |
| **WP019** | chr1 | 149073703 | C | T | non-protein affecting | intron | p(?) |
| **WP019** | chr17 | 76736877 | G | T | **driver (oncodriveMUT)** | missense | p.Pro95His |
| **WP019** | chr2 | 113117932 | G | T | non-protein affecting | 5_prime_UTR | p(?) |
| **WP019** | chr4 | 2341569 | G | C | passenger | missense | p.Pro76Arg |
| **WP019** | chr6 | 158130554 | A | C | non-protein affecting | intron | p(?) |
| **WP019** | chr7 | 111783014 | G | A | non-protein affecting | intron | p(?) |
| **WP048** | chr1 | 114716123 | C | T | **driver (boostDM)** | missense | p.Gly13Asp (G13D) |
| **WP048** | chr9 | 5073770 | G | T | passenger (oncodriveMUT)* | missense | p.Val617Phe (V617F) |
| **WP058** | chr12 | 57185563 | T | G | passenger | missense | p.Cys2166Gly |
| **WP058** | chr15 | 28272094 | A | G | non-protein affecting | intron | p(?) |
| **WP058** | chr15 | 43206304 | T | G | non-protein affecting | intron | p(?) |
| **WP058** | chr19 | 12943751 | (del) | - | **driver (oncodriveMUT)** | frameshift | p.Leu367ThrfsTer46 |
| **WP058** | chr2 | 105892889 | T | G | non-protein affecting | intron | p(?) |
| **WP058** | chr20 | 32434638 | - | G | **driver (oncodriveMUT)** | frameshift | p.Gly646TrpfsTer12 |
| **WP058** | chr7 | 124892275 | T | C | passenger | missense | p.Lys39Glu |
| **WP068** | chr1 | 2498679 | T | G | non-protein affecting | intron | p(?) |
| **WP068** | chr11 | 17388932 | A | C | non-protein affecting | upstream_gene | p(?) |
| **WP068** | chr11 | 56290914 | C | T | passenger | missense | p.Arg50His |
| **WP068** | chr14 | 30877438 | T | G | non-protein affecting | intron | p(?) |
| **WP068** | chr19 | 49441413 | T | C | non-protein affecting | 5_prime_UTR | p(?) |
| **WP068** | chr21 | 37147717 | A | C | non-protein affecting | intron | p(?) |
| **WP068** | chr4 | 54733155 | A | T | **driver (boostDM)** | missense | p.Asp816Val (D816V) |
| **WP068** | chr4 | 105276128 | T | C | **driver (oncodriveMUT)** | missense | p.Ile1894Thr |
| **WP068** | chr4 | 155903714 | G | T | non-protein affecting | 5_prime_UTR | p(?) |
| **WP068** | chr7 | 47837003 | G | A | passenger | missense | p.Thr1954Met |
| **WP068** | chr7 | 151045303 | C | T | passenger | missense | p.Pro721Leu |

> \*Nota: Para a amostra WP048, a variante V617F (JAK2) foi classificada como "passenger" pelo oncodriveMUT, mas é uma variante canônica de mielofibrose/trombocitemia.

## Passo 9: Verificando o biomarcador (arquivo biomarkers.tsv)
```Python
import pandas as pd
pd.read_csv('/content/biomarkers.tsv',sep='\t',index_col=False, engine= 'python')
````
### Predição de Resposta a Drogas (Biomarkers)

| Sample ID | Alteration | Drug | Disease | Response | Evidence Source |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **WP017** | NRAS (G12S) | Cetuximab | Colorectal adenocarcinoma | **Resistant** | PMID:24024839... |
| **WP017** | NRAS (G12S) | Panitumumab | Colorectal adenocarcinoma | **Resistant** | FDA guidelines |
| **WP017** | KRAS wildtype | Panitumumab | Colorectal adenocarcinoma | Responsive | FDA / Drug Label |
| **WP017** | KRAS wildtype | Cetuximab | Colorectal adenocarcinoma | Responsive | PMID: 19339720 |
| **WP017** | EGFR/ALK wt | Atezolizumab; Bevacizumab | NSCLC | Responsive | FDA Label |
| **WP048** | NRAS (G13D) | Temozolomide | Cutaneous melanoma | Responsive | CIViC (Link) |
| **WP048** | NRAS (G13D) | Cetuximab | Colorectal adenocarcinoma | **Resistant** | CIViC (Link) |
| **WP048** | PIK3CA wt | Aspirin | Colorectal adenocarcinoma | Responsive | CIViC (Link) |
| **WP048** | TP53 wt | Cetuximab, Oxaliplatin | Colorectal adenocarcinoma | Responsive | CIViC (Link) |
| **WP048** | TP53 wt | Adjuvant Chemotherapy | NSCLC | Responsive | CIViC (Link) |
## Passo 10: Baixar o Google Drive, pra poder salvar os arquivos. (Pra não precisar ficar rodando tudo do 0 pra ver os arquivos)
```Python
from google.colab import drive
drive.mount('/content/drive')
```
- Usar o passo 10 apenas no Google Collab
### Passo 10.1
```Bash
MeuDrive="/content/drive/MyDrive/PGBIOAGMAV/Somaticos/AulaPratica"
mkdir -p $MeuDrive # Criar a pastinha no Drive
cp /content/alterations.tsv $MeuDrive #alterações
cp /content/biomarkers.tsv $MeuDrive # biomarcadores
cp /content/df_WPALL-cgi.txt $MeuDrive # amostra
cp /content/input01.tsv $MeuDrive # id
cp /content/sample01.zip $MeuDrive # amostra
cp /content/summary.txt $MeuDrive # sumario
```

## Passo 11: Deletando o Job no CGI (Opcional)
```Python
import requests
job_id = id individual

headers = {'Authorization': 'kaneka4850@gmail.com Seu_Token'} # permissões do CGI
r = requests.delete('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json() 

print("Job deletado com sucesso") 
```
## EXTRA: (Gerar um HTML pra não ter que ficar lendo todo o biomarker, pois ele é muito grande)
```Python
import pandas as pd
import json
from google.colab import files

# 1. Carregamento dos Dados com Pandas
try:
    df_alt = pd.read_csv('alterations.tsv', sep='\t')
    df_bio = pd.read_csv('biomarkers.tsv', sep='\t')

    print("✅ Arquivos carregados com sucesso!")
except FileNotFoundError:
    print("❌ Erro: Por favor, faça o upload dos arquivos")
    raise

# 2. Limpeza e Pré-processamento
# Substitui valores vazios (NaN) por string vazia para compatibilidade com JSON/HTML
df_alt = df_alt.fillna('')
df_bio = df_bio.fillna('')


# 3. Conversão para JSON
# O Pandas faz isso nativamente, facilitando muito a injeção no HTML
json_alt = df_alt.to_json(orient='records')
json_bio = df_bio.to_json(orient='records')

# 4. Template HTML (A estrutura visual do relatório)
html_template = """
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Relatório Genômico (Gerado via Pandas)</title>
    <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.11.5/css/jquery.dataTables.css">
    <style>
        :root { --bg: #f4f7f6; --card: #fff; --text: #333; --accent: #3498db; }
        [data-theme="dark"] { --bg: #1a1a1a; --card: #2d2d2d; --text: #e0e0e0; }
        body { font-family: sans-serif; background: var(--bg); color: var(--text); padding: 20px; }
        .container { max-width: 1400px; margin: 0 auto; }
        .card { background: var(--card); padding: 20px; border-radius: 8px; margin-bottom: 20px; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        h2 { border-left: 4px solid var(--accent); padding-left: 10px; margin-top: 0; }

        /* Badges */
        .badge { padding: 4px 8px; border-radius: 4px; font-weight: bold; font-size: 0.85em; }
        .oncogenic { background: #ffebee; color: #c0392b; border: 1px solid #c0392b; }
        .passenger { background: #f0f2f5; color: #7f8c8d; border: 1px solid #bdc3c7; }
        .responsive { background: #e8f5e9; color: #27ae60; border: 1px solid #27ae60; }
        .resistant { background: #ffebee; color: #c0392b; border: 1px solid #c0392b; }
        .evidence { display: inline-block; width: 25px; height: 25px; text-align: center; background: var(--accent); color: white; border-radius: 50%; line-height: 25px; }

        /* Controls */
        .controls { display: flex; gap: 15px; margin-bottom: 20px; align-items: center; background: var(--card); padding: 15px; border-radius: 8px; }
        select { padding: 8px; }

        /* Dark Mode overrides for DataTables */
        [data-theme="dark"] .dataTables_wrapper { color: #a0a0a0; }
        [data-theme="dark"] table.dataTable tbody tr { background-color: var(--card); }
    </style>
</head>
<body>

<div class="container">
    <div class="controls">
        <h1>🧬 Dashboard Genômico</h1>
        <select id="sampleSelect"><option value="">Todas as Amostras</option></select>
        <label><input type="checkbox" id="relevantOnly"> Apenas Relevantes</label>
        <button onclick="toggleTheme()">Alternar Tema</button>
    </div>

    <div class="card">
        <h2>🔬 Variantes (Alterations)</h2>
        <table id="altTable" class="display" style="width:100%">
            <thead><tr><th>Sample</th><th>Gene</th><th>Protein</th><th>Type</th><th>Summary</th><th>Prediction</th></tr></thead>
        </table>
    </div>

    <div class="card">
        <h2>💊 Biomarcadores (Biomarkers)</h2>
        <table id="bioTable" class="display" style="width:100%">
            <thead><tr><th>Sample</th><th>Alteration</th><th>Drug</th><th>Disease</th><th>Response</th><th>Evidence</th></tr></thead>
        </table>
    </div>
</div>

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://cdn.datatables.net/1.11.5/js/jquery.dataTables.js"></script>
<script>
    // DADOS INJETADOS PELO PYTHON/PANDAS
    const altData = __ALT_JSON__;
    const bioData = __BIO_JSON__;

    let altTable, bioTable;

    function init() {
        // Popular Select
        const samples = [...new Set([...altData.map(d => d['SAMPLE']), ...bioData.map(d => d['Sample ID'])])].sort();
        const select = document.getElementById('sampleSelect');
        samples.forEach(s => { if(s) select.innerHTML += `<option value="${s}">${s}</option>`; });

        // Tabela Variantes
        altTable = $('#altTable').DataTable({
            data: altData,
            columns: [
                { data: 'SAMPLE' }, { data: 'CGI-Gene' }, { data: 'CGI-Protein Change' }, { data: 'CGI-Type' },
                { data: 'CGI-Oncogenic Summary', render: d => {
                    const isOnco = (d||'').toLowerCase().includes('oncogenic') && !(d||'').toLowerCase().includes('non-oncogenic');
                    return `<span class="badge ${isOnco ? 'oncogenic' : 'passenger'}">${d}</span>`;
                }},
                { data: 'CGI-Oncogenic Prediction' }
            ]
        });

        // Tabela Biomarcadores
        bioTable = $('#bioTable').DataTable({
            data: bioData,
            columns: [
                { data: 'Sample ID' }, { data: 'Alterations' }, { data: 'Drugs' }, { data: 'Diseases' },
                { data: 'Response', render: d => {
                    const isRes = (d||'').toLowerCase().includes('resistant');
                    return `<span class="badge ${isRes ? 'resistant' : 'responsive'}">${d}</span>`;
                }},
                { data: 'Evidence', render: d => `<span class="evidence">${d}</span>` }
            ]
        });

        // Listeners
        $('#sampleSelect, #relevantOnly').on('change', () => { altTable.draw(); bioTable.draw(); });

        // Filtro Customizado
        $.fn.dataTable.ext.search.push((settings, data, idx, row) => {
            const sample = $('#sampleSelect').val();
            const relevant = $('#relevantOnly').is(':checked');

            // Filtro de Amostra
            const rowSample = settings.nTable.id === 'altTable' ? row['SAMPLE'] : row['Sample ID'];
            if (sample && rowSample !== sample) return false;

            // Filtro de Relevância
            if (relevant) {
                if (settings.nTable.id === 'altTable') {
                    const summ = (row['CGI-Oncogenic Summary']||'').toLowerCase();
                    const pred = (row['CGI-Oncogenic Prediction']||'').toLowerCase();
                    if (!pred.includes('driver') && !(summ.includes('oncogenic') && !summ.includes('non-oncogenic'))) return false;
                } else {
                    if (row['Evidence'] !== 'A' && row['Evidence'] !== 'B') return false;
                }
            }
            return true;
        });
    }

    function toggleTheme() {
        const current = document.documentElement.getAttribute('data-theme');
        document.documentElement.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
    }

    $(document).ready(init);
</script>
</body>
</html>
"""

# 5. Injeção e Salvamento
# Substituímos os placeholders __ALT_JSON__ e __BIO_JSON__ pelos dados reais do Pandas
final_html = html_template.replace('__ALT_JSON__', json_alt).replace('__BIO_JSON__', json_bio)

output_filename = 'relatorio_pandas_genomica.html'
with open(output_filename, 'w', encoding='utf-8') as f:
    f.write(final_html)

print(f"✅ Arquivo '{output_filename}' gerado com sucesso!")

# 6. Download Automático
files.download(output_filename)
```

# PASSO DE CRIAÇÃO DOS TIERS
Visando ter umm segundo critério de analise das variantes somáticas, usaremos agora criação de tiers, e no final realizar a comparação das amostras analisadas pelo cgi x critérios feitos aqui

## 1.0 Criação das tabelas de Tiers (No ambiente do collab)
Para garantir a automatização do processo, o script abaixo ira verificar todas as amostras (formato VCF) e realizar a conversão de tabelas no formato tsv, lembrando que deve ser instalado a ferramanta BCFtools para funciona!

```bash
!sudo apt install bcftools
```
Após a instalação do pacote, é possivel a rodar o script, lembrando que ele roda todos os arquivos que estiverem no diretório (pasta), então certifique-se de deixar todos os vcfs a serem analisados de uma unica vez
```bash
# Define o diretório onde os arquivos de entrada estão
DIR_INPUT="lmabrasil-hg38/vep_output"

# Define o diretório onde os arquivos de saída serão salvos
DIR_OUTPUT="outputs"

# Cria o diretório de saída (o -p garante que não dê erro se já existir)
mkdir -p "$DIR_OUTPUT"

# Inicia o loop
for ARQUIVO in ${DIR_INPUT}/liftOver_WP*_hg19ToHg38.vep.vcf; do

    # 1. Extrair o ID da amostra
    NOME_BASE=$(basename "$ARQUIVO")
    SAMPLE_ID=$(echo "$NOME_BASE" | cut -d'_' -f2)

    # Define o caminho completo do arquivo de saída (agora dentro da pasta outputs)
    OUTPUT_FILE="${DIR_OUTPUT}/liftOver_${SAMPLE_ID}_hg19ToHg38.vep.tsv"

    echo "Processando amostra: ${SAMPLE_ID} -> salvando em ${OUTPUT_FILE}"

    # 2. Criar o cabeçalho
    bcftools +split-vep -l "$ARQUIVO" | \
    cut -f2 | \
    tr '\n\r' '\t' | \
    awk '{print("CHROM\tPOS\tREF\tALT\t"$0"FILTER\tTumorID\tGT\tDP\tAD\tAF\tNormalID\tNGT\tNDP\tNAD\tNAF")}' \
    > "$OUTPUT_FILE"

    # 3. Adicionar as variantes
    bcftools +split-vep \
    -f '%CHROM\t%POS\t%REF\t%ALT\t%CSQ\t%FILTER\t[%SAMPLE\t%GT\t%DP\t%AD\t%AF\t]\n' \
    -i 'FMT/DP>=20 && FMT/AF>=0.1' -d -A tab "$ARQUIVO" \
    -p x >> "$OUTPUT_FILE"

done

echo "Processamento finalizado. Arquivos salvos em: ${DIR_OUTPUT}/"
```
note que o script vai filtrar por nome do arquivo, então verifique sempre os nomes dos vcfs
## 2.0 Definição dos tiers de acordo com cada amostra
Agora o próximo passo é fazer a definição dos tiers de cada amostra, sendo dividido em tier 1, tier 2 e tier 3, lembrando que o script analisa por amostra individualmente, então pode ter variaçõoes de desempenho conforme o número de amostras
```python
import pandas as pd
import numpy as np
import re
import requests
import glob
import os

# =========================
# CONFIG
# =========================
INPUT_PATTERN = "outputs/liftOver_WP*_hg19ToHg38.vep.tsv"
OUTPUT_DIR    = "outputs"

# Caminho local (se você clonou o repo)
LOCAL_DRIVER_PATH = "lmabrasil-hg38/hpo/Clonal_Hematopoiesis_driver_genes.txt"

# URL de backup (caso não tenha o arquivo local)
DRIVER_WEB_URL = "https://raw.githubusercontent.com/renatopuga/lmabrasil-hg38/refs/heads/main/hpo/Clonal_Hematopoiesis_driver_genes.txt"

# =========================
# FUNÇÕES
# =========================
def smart_read_tsv(path: str) -> pd.DataFrame:
    with open(path, "r", encoding="utf-8", errors="replace") as f:
        header = f.readline().rstrip("\n")
        cols = header.split("\t")
        n = len(cols)
        rows = []
        for line in f:
            line = line.rstrip("\n")
            parts = line.split("\t")
            if len(parts) > n:
                parts = parts[:n-1] + ["\t".join(parts[n-1:])]
            elif len(parts) < n:
                parts = parts + [""] * (n - len(parts))
            rows.append(parts)
    return pd.DataFrame(rows, columns=cols)

def pick_first_existing_col(df: pd.DataFrame, candidates):
    for c in candidates:
        if c in df.columns: return c
    return None

def parse_percent_vaf(x):
    if x is None: return np.nan
    s = str(x).strip()
    if s in {"", ".", "NA", "NaN", "nan", "None"}: return np.nan
    s = s.replace(",", ".").replace("%", "")
    try:
        v = float(s)
    except:
        return np.nan
    return v * 100.0 if v <= 1.0 else v

def filter_is_pass(x):
    return str(x).strip().upper() == "PASS"

def load_driver_genes(local_path, web_url):
    content = ""
    # 1. Tenta ler localmente
    if os.path.exists(local_path):
        print(f"Carregando genes do arquivo LOCAL: {local_path}")
        try:
            with open(local_path, "r") as f:
                content = f.read()
        except Exception as e:
            print(f"Erro ao ler local: {e}. Tentando web...")

    # 2. Se não conseguiu local, tenta web
    if not content:
        print(f"Baixando lista de genes da WEB: {web_url}")
        try:
            r = requests.get(web_url, timeout=30)
            r.raise_for_status()
            content = r.text
        except Exception as e:
            print(f"ERRO CRÍTICO: Não foi possível carregar os genes nem local nem web. {e}")
            return set()

    # Processa o texto
    tokens = re.split(r"[\s,;]+", content.strip())
    genes = {t.strip().upper() for t in tokens if t.strip() and not t.startswith("#")}
    return genes

# =========================
# EXECUÇÃO PRINCIPAL
# =========================

# 1. Carrega Driver Genes (Híbrido)
driver_genes = load_driver_genes(LOCAL_DRIVER_PATH, DRIVER_WEB_URL)

print(f"Total de Driver Genes carregados: {len(driver_genes)}")
if len(driver_genes) == 0:
    print("!!! ALERTA: Lista de genes vazia. Todos os resultados serão Tier 3. Verifique os caminhos !!!")

# 2. Busca arquivos
file_list = glob.glob(INPUT_PATTERN)
print(f"Encontrados {len(file_list)} arquivos para processar.\n")

# 3. Loop
for input_tsv in file_list:
    filename = os.path.basename(input_tsv)
    parts = filename.split('_')

    # Pega o ID (WPxxx)
    sample_id = parts[1] if len(parts) >= 2 else "UNKNOWN"
    output_tsv = os.path.join(OUTPUT_DIR, f"{sample_id}-tier.tsv")

    print(f"--> Processando: {sample_id}")

    try:
        df = smart_read_tsv(input_tsv)

        # Mapeamento
        gene_col     = pick_first_existing_col(df, ["SYMBOL", "Gene", "GENE", "HGNC"])
        filter_col   = pick_first_existing_col(df, ["FILTER", "Filter"])
        dp_col       = pick_first_existing_col(df, ["DP", "DP_tumor", "TUMOR_DP", "DEPTH"])
        vaf_col      = pick_first_existing_col(df, ["VAF_tumor", "VAF", "AF", "TUMOR_AF"])
        existing_col = pick_first_existing_col(df, ["Existing_variation", "ExistingVariation", "existing_variation"])

        missing = [("SYMBOL", gene_col), ("FILTER", filter_col), ("DP", dp_col), ("VAF/AF", vaf_col), ("Existing_variation", existing_col)]
        missing = [name for name, col in missing if col is None]

        if missing:
            print(f"   [PULANDO] Colunas faltando: {missing}")
            continue

        # Filtros
        df["_GENE_UP"] = df[gene_col].astype(str).str.strip().str.upper()
        df["_PASS"]    = df[filter_col].apply(filter_is_pass)
        df["_DP"]      = pd.to_numeric(df[dp_col].astype(str).str.replace(",", ".", regex=False), errors="coerce").fillna(0).astype(int)
        df["_VAF_PCT"] = df[vaf_col].apply(parse_percent_vaf)
        df["_EXIST"]   = df[existing_col].astype(str)

        base  = (df["_PASS"]) & (df["_DP"] >= 20) & (df["_VAF_PCT"] >= 10)

        # Regras de Classificação
        tier1 = base & (df["_GENE_UP"].isin(driver_genes))
        tier2 = base & (~tier1) & (df["_EXIST"].str.contains(r"\bCOSV\b", flags=re.IGNORECASE, regex=True))
        tier3 = ~(tier1 | tier2)

        df["Tier"] = np.select([tier1, tier2, tier3], ["Tier 1", "Tier 2", "Tier 3"], default="Tier 3")

        df.drop(columns=[c for c in df.columns if c.startswith("_")], inplace=True)
        df.to_csv(output_tsv, sep="\t", index=False)

        counts = df["Tier"].value_counts().to_dict()
        print(f"    Salvo: {output_tsv} | Tiers: {counts}")

    except Exception as e:
        print(f"    [ERRO] {sample_id}: {e}")

print("\nConcluído.")
```
## 3.0 Criação dos diagramas de venn de acordo com cada amostra analisada
Após realizar a analise de todas as amostras, o proximo passo é realizar o comparativo com as analises feitas no GCI, lembrando que para que esse script funcione, ele espera além dos inputs (que são or arquivos gerados no script acima), uma referencia, no caso o arquivo alterations.tsv, sendo obtido no processo final do CGI, por conta disso, verifique de gerar o arquivo, caso contrario tera erro.

```python
import pandas as pd
import re
import matplotlib.pyplot as plt
from matplotlib_venn import venn2, venn2_circles
import glob
import os

# =========================
# CONFIG
# =========================
ALTERATIONS_TSV = "/content/alterations.tsv"
INPUT_PATTERN   = "outputs/*-tier.tsv"
OUTPUT_DIR      = "outputs"

# =========================
# FUNÇÕES AUXILIARES
# =========================
def pick_first_existing_col(df, candidates):
    for c in candidates:
        if c in df.columns:
            return c
    return None

def norm_chr(x):
    s = str(x).strip()
    if not s:
        return s
    return s if s.lower().startswith("chr") else "chr" + s

def make_var_id(df, chr_col, pos_col, ref_col, alt_col):
    return (
        df[chr_col].astype(str).map(norm_chr) + ":" +
        df[pos_col].astype(str) + ":" +
        df[ref_col].astype(str) + ":" +
        df[alt_col].astype(str)
    )

# ========================================================
# 1) CARREGAR DRIVER SET
# ========================================================
print("Carregando base de Drivers (alterations.tsv)...")

try:
    alt = pd.read_csv(ALTERATIONS_TSV, sep="\t", dtype=str)

    chr_col = pick_first_existing_col(alt, ["CHR", "CHROM", "#CHROM", "CHROMOSOME"])
    pos_col = pick_first_existing_col(alt, ["POS", "POSITION"])
    ref_col = pick_first_existing_col(alt, ["REF"])
    alt_col = pick_first_existing_col(alt, ["ALT"])
    pred_col = pick_first_existing_col(alt, ["CGI-Oncogenic Prediction", "CGI-Oncogenic Summary"])

    missing = [("CHR", chr_col), ("POS", pos_col), ("REF", ref_col), ("ALT", alt_col), ("DriverColumn", pred_col)]
    missing = [name for name, col in missing if col is None]
    if missing:
        raise ValueError(f"Faltando colunas no alterations.tsv: {missing}")

    alt["VAR_ID"] = make_var_id(alt, chr_col, pos_col, ref_col, alt_col)

    # Filtro de driver
    is_driver = alt[pred_col].fillna("").astype(str).str.contains(r"\bdriver\b", flags=re.IGNORECASE, regex=True)
    set_driver = set(alt.loc[is_driver, "VAR_ID"])

    print(f"Total de Drivers identificados na base: {len(set_driver)}")

except Exception as e:
    print(f"ERRO CRÍTICO ao ler alterations.tsv: {e}")
    set_driver = set()

# ========================================================
# 2) LOOP POR TODAS AS AMOSTRAS
# ========================================================

file_list = glob.glob(INPUT_PATTERN)
print(f"\nEncontrados {len(file_list)} arquivos Tier para analisar.\n")

for tier_file in file_list:
    filename = os.path.basename(tier_file)
    sample_id = filename.split('-')[0]

    print(f"--> Analisando: {sample_id}")

    try:
        tiers = pd.read_csv(tier_file, sep="\t", dtype=str)

        t_chr = pick_first_existing_col(tiers, ["CHROM", "#CHROM", "CHR", "Chrom"])
        t_pos = pick_first_existing_col(tiers, ["POS", "Position", "pos"])
        t_ref = pick_first_existing_col(tiers, ["REF", "Ref"])
        t_alt = pick_first_existing_col(tiers, ["ALT", "Alt"])

        if not all([t_chr, t_pos, t_ref, t_alt]):
            print(f"   [PULANDO] {sample_id}: Colunas de coordenadas não encontradas.")
            continue

        tiers["VAR_ID"] = make_var_id(tiers, t_chr, t_pos, t_ref, t_alt)
        set_t1 = set(tiers.loc[tiers["Tier"].astype(str) == "Tier 1", "VAR_ID"])

        # ----------------------------
        # 3) Venn & Salvamento (ESTILO NOVO)
        # ----------------------------

        # Aumentar tamanho da figura para melhor resolução
        plt.figure(figsize=(8, 8))

        # Cores profissionais: Azul Petróleo (Database) e Laranja Ouro (Amostra)
        # Alpha define a transparência
        colors = ('#006ba4', '#ff800e')

        # Cria o Venn
        v = venn2([set_driver, set_t1],
                  set_labels=("Banco de Dados\nCGI Drivers", f"Amostra {sample_id}\n(Tier 1)"),
                  set_colors=colors,
                  alpha=0.7)

        # Adiciona bordas sólidas aos círculos para acabamento premium
        c = venn2_circles([set_driver, set_t1], linestyle='-', linewidth=1, color='grey')

        # CUSTOMIZAÇÃO FINA DAS FONTES
        try:
            # Aumentar fonte dos números dentro do gráfico
            for text in v.subset_labels:
                if text:
                    text.set_fontsize(22) # Tamanho do número
                    text.set_fontweight('bold')

            # Aumentar fonte dos Labels (títulos dos círculos)
            for text in v.set_labels:
                if text:
                    text.set_fontsize(14)
        except:
            pass # Caso algum set esteja vazio, evita erro

        # Título mais profissional e descritivo
        plt.title(f"Interseção de Variantes: {sample_id} vs CGI", fontsize=18, pad=20)

        # Salvar imagem em ALTA RESOLUÇÃO (300 DPI) e cortando bordas brancas (bbox_inches)
        img_path = os.path.join(OUTPUT_DIR, f"venn_{sample_id}_HQ.png")
        plt.savefig(img_path, dpi=300, bbox_inches='tight')
        plt.close()

        # Salvar txts
        intersection = sorted(set_driver & set_t1)
        txt_path = os.path.join(OUTPUT_DIR, f"venn_intersection_{sample_id}.txt")
        pd.Series(intersection).to_csv(txt_path, index=False, header=False)

        print(f"   Tier 1 encontrados: {len(set_t1)}")
        print(f"   Em comum (Interseção): {len(intersection)}")
        print(f"   Salvo imagem HQ: {img_path}")

    except Exception as e:
        print(f"   [ERRO] Falha ao processar {sample_id}: {e}")

print("\nAnálise de Venn finalizada para todas as amostras.")
```

<img src="https://github.com/user-attachments/assets/814340f4-9feb-4a52-9df9-d10ea2cc9a99" width="268" height="268" alt="image" />

