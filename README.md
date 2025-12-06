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

l
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
head df_WP048-cgi.txt
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
with open('sample01.final', 'wb') as fd:
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
<img src="https://github.com/user-attachments/assets/814340f4-9feb-4a52-9df9-d10ea2cc9a99" width="268" height="268" alt="image" />

