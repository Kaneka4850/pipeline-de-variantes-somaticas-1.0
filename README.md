# Pipeline somatico - DO VCF Anotado ao CGI (Câncer Genome Interpreter)

# 1 - Como obter o Token no CGI:
# https://www.cancergenomeinterpreter.org/rest_api

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
job_id = # Job-Id individual

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
job_id = # Coloque seu Job_Id individual aqui

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
job_id = # id individual

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
cp /content/df_WP048-cgi.txt $MeuDrive # amostra
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

<img src="https://github.com/user-attachments/assets/814340f4-9feb-4a52-9df9-d10ea2cc9a99" width="268" height="268" alt="image" />

