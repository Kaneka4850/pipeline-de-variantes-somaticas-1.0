# Pipeline somatico - DO VCF Anotado ao CGI (Câncer Genome Interpreter)

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
<img width="1212" height="188" alt="image" src="https://github.com/user-attachments/assets/9c165a86-59cd-4132-b39f-a35f2d851f45" />
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
## Passo 4: Verificação do Status do Job_ID, lembrando que o job_id é individual
- Obs: Coloca o Token e seu Job_id aqui, não copia sem mudar não! :)
```Python
import requests
job_id = # Job-Id individual

headers = {'Authorization': 'kaneka4850@gmail.com Seu_TOKEN'}
r = requests.get('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json()
```

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
## Passo 7: 7. Descompactar o .zip utilizando unzip.

```Bash
unzip sample01.zip # comando para descompactar
```

## Passo 8: Verificando o resultado das alterações: arquivo alterations.tsv
```Python
# BAIXA A MERDA DO PANDAS ANTES
import pandas as pd
pd.read_csv('/content/alterations.tsv',sep='\t',index_col=False, engine= 'python')
```
## Passo 9: Verificando o biomarcador (arquivo biomarkers.tsv)
```Python
import pandas as pd
pd.read_csv('/content/biomarkers.tsv',sep='\t',index_col=False, engine= 'python')
````
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
 # job_id ="314a1ffb93d1bb435a3e"  id individual

headers = {'Authorization': 'kaneka4850@gmail.com c617eeb5f78dc078813f'} # permissões do CGI
r = requests.delete('https://www.cancergenomeinterpreter.org/api/v1/%s' % job_id, headers=headers)
r.json() 

print("Job deletado com sucesso") 
```
