# Pipeline somatico - DO VCF Anotado ao CGI (Câncer Genome Interpreter)

# 1 - Como obter o Token no CGI:
# https://www.cancergenomeinterpreter.org/rest_api

## Passo 1: Clonar o GitHub do LMA Brasil, seguindo o comando para que ele funcione
```bash
git clone https://github.com/renatopuga/lmabrasil-hg38.git
```
## Passo 2: Usar o comando para filtrar o arquivo, deixando ele aceitavel para o GCI
```bash
cut -f1-4 /content/lmabrasil-hg38/vep_output/liftOver_WP048_hg19ToHg38.vep.filter.tsv | sed -e "s/CHROM/CHR/g"  > df_WP048-cgi.txt
head df_WP048-cgi.txt
```
### Output esperado:
<img width="326" height="65" alt="image" src="https://github.com/user-attachments/assets/791aeca8-df4a-46a7-bfc9-3826b0d6591b" />

- Resultado esperado: fonte Google Collab

## Passo 3: Enviando um request "requisição" a API do CGI
Esse código em python permite realizar a requisição a API via código PYTHON, também tem a opção via bash, mas para fins didaticos, sera explicado via python
- Obs: No lugar de Seu_TOKEN, colocar o Token obtido no CGI
```python
import requests
# cabeçalho 
headers = {'Authorization': 'kaneka4850@gmail.com Seu_TOKEN'}
payload = {'cancer_type': 'HEMATO', 'title': 'Somatic MF', 'reference': 'hg38'}
# requisição
r = requests.post('https://www.cancergenomeinterpreter.org/api/v1',
                headers=headers,
                files={
                        'mutations': open('/content/df_WP048-cgi.txt', 'rb')
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
with open('sample01.zip', 'wb') as fd:
    fd.write(r._content)
```
- Esse não vai gerar output, então não fica triste, não dando erro é o que importa
## Passo 7: 7. Descompactar o .zip utilizando unzip.

```Bash
unzip sample01.zip # comando para descompactar
```


<img width="339" height="98" alt="image" src="https://github.com/user-attachments/assets/fa186ee0-4347-408f-ba37-bc728bb5270a" />



Resultado esperado:

<img width="280" height="206" alt="image" src="https://github.com/user-attachments/assets/bc7b8e66-8ff4-494d-946c-db956e0deef1" />


## Passo 8: Verificando o resultado das alterações: arquivo alterations.tsv
```Python
import pandas as pd
pd.read_csv('/content/alterations.tsv',sep='\t',index_col=False, engine= 'python')
```
| Input ID | CHROMOSOME | POSITION | REF | ALT | CHR | POS | ALT_TYPE | STRAND | CGI-Sample ID | CGI-Oncogenic Prediction | CGI-External oncogenic annotation | CGI-Mutation | CGI-Consequence | CGI-Transcript | CGI-STRAND | CGI-Type | CGI-HGVS | CGI-HGVSc | CGI-HGVSp |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| input01_1 | 1 | 114716123 | C | T | chr1 | 114716123 | snp | + | input01 | driver (boostDM: non-tissue-specific model) | cgi,oncokb,clinvar:13901 | chr1:114716123 C>T | missense_variant | ENST00000369535 | + | SNV | ENST00000369535:c.38G>A;p.(Gly13Asp);p.(G13D) | ENST00000369535.5:c.38G>A | ENSP00000358548.4:p.Gly13Asp |
| input01_2 | 9 | 5073770 | G | T | chr9 | 5073770 | snp | + | input01 | passenger (oncodriveMUT) | cgi,oncokb,clinvar:14662 | chr9:5073770 G>T | missense_variant | ENST00000381652 | + | SNV | ENST00000381652:c.1849G>T;p.(Val617Phe);p.(V617F) | ENST00000381652.4:c.1849G>T | ENSP00000371067.4:p.Val617Phe |

## Passo 9: Verificando o biomarcador (arquivo biomarkers.tsv)
```Python
import pandas as pd
pd.read_csv('/content/biomarkers.tsv',sep='\t',index_col=False, engine= 'python')
````
| Sample ID | Alterations | Biomarker | Drugs | Diseases | Response | Evidence | Match | Source | BioM | Resist. | Tumor type |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| input01 | NRAS (G13D) | NRAS (12,13,59,61,117,146) | Cetuximab (EGFR mAb inhibitor) | Colorectal adenocarcinoma | Resistant | A | NO | PMID:24024839; PMID:20619739; PMID:23325582 | complete | NaN | COREAD |
| input01 | NRAS (G13D) | NRAS (12,13,59,61,117,146) | Panitumumab (EGFR mAb inhibitor) | Colorectal adenocarcinoma | Resistant | A | NO | FDA guidelines | complete | NaN | COREAD |
| input01 | JAK2 (V617F) | JAK2 (V617F) | Ruxolitinib (JAK inhibitor) | Myelofibrosis | Responsive | A | YES | FDA | complete | NaN | MY |
| input01 | KRAS wildtype | KRAS wildtype | Panitumumab (EGFR mAb inhibitor) | Colorectal adenocarcinoma | Responsive | A | NO | FDA https://www.accessdata.fda.gov... | complete | 65.0 | COREAD |
| input01 | KRAS wildtype | KRAS wildtype | Cetuximab (EGFR mAb inhibitor) | Colorectal adenocarcinoma | Responsive | A | NO | PMID: 19339720 | complete | 22.0 | COREAD |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| input01 | NRAS (G13D) | NRAS Q61R | Temozolomide | Cutaneous melanoma | Responsive | C | NO | https://civicdb.org/links/evidence_items/23 | only alteration type | NaN | CM |
| input01 | NRAS (G13D) | NRAS Q61R | Cetuximab | Colorectal adenocarcinoma | Resistant | C | NO | https://civicdb.org/links/evidence_items/2184 | only alteration type | NaN | COREAD |
| input01 | PIK3CA wildtype | PIK3CA Wildtype | Aspirin | Colorectal adenocarcinoma | Responsive | B | NO | https://civicdb.org/links/evidence_items/2038 | complete | NaN | COREAD |
| input01 | TP53 wildtype | TP53 Wildtype | Cetuximab, Oxaliplatin, Capecitabine | Colorectal adenocarcinoma | Responsive | B | NO | https://civicdb.org/links/evidence_items/875 | complete | NaN | COREAD |
| input01 | TP53 wildtype | TP53 Wildtype | Adjuvant Chemotherapy | Non-small cell lung | Responsive | B | NO | https://civicdb.org/links/evidence_items/1149 | complete | NaN | NSCLC |
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

