# DroneVision 🛰️

Aplicação Web que integra a **Visão Computacional da Azure (Azure AI Vision)** para
analisar imagens capturadas por um drone de monitoramento aéreo. O operador envia
uma fotografia e recebe uma **descrição gerada por Inteligência Artificial**, além
das **tags** e **objetos** identificados na cena.

> **GS 2º Semestre — Cloud Computing & Machine as a Service (MaaS) — FIAP 5EMR**
>
> - Marcelo Alves da Silva Júnior — **RM93624**
> - João Pedro Leça Ferreira — **RM565431**

---

## 🧩 Arquitetura

```
  Operador
     │  (envia imagem pelo navegador)
     ▼
┌─────────────────────────────┐        ┌──────────────────────────────┐
│  Web App (Flask / Python)   │  HTTPS │   Azure AI Vision             │
│  Azure App Service (Linux)  │ ─────▶ │   (Computer Vision - Analyze) │
│  app-dronevision-rm93624    │ ◀───── │   aiv-dronevision-rm93624     │
└─────────────────────────────┘  JSON  └──────────────────────────────┘
     │  (descrição + tags + objetos)
     ▼
  Resultado exibido na página
```

### Recursos criados na Azure (padrão de nomenclatura da GS)

| Recurso           | Nome                        |
|-------------------|-----------------------------|
| Resource Group    | `rg-dronevision-rm93624`    |
| Azure AI Vision   | `aiv-dronevision-rm93624`   |
| App Service Plan  | `plan-dronevision-rm93624`  |
| Web App           | `app-dronevision-rm93624`   |

---

## ☁️ Como implantar (Azure Cloud Shell — Bash)

Toda a infraestrutura é criada por **script Shell usando a Azure CLI**, conforme
exigido. A aplicação **não** roda em localhost — é publicada no Azure App Service.

1. Suba este projeto para o **GitHub**.
2. Abra o **[Azure Cloud Shell](https://shell.azure.com)** no modo **Bash**.
3. Clone o repositório e entre na pasta:
   ```bash
   git clone <URL_DO_SEU_REPOSITORIO>
   cd <pasta-do-repositorio>
   ```
4. Dê permissão de execução e rode o script:
   ```bash
   chmod +x implantar-rm93624.sh
   ./implantar-rm93624.sh
   ```
5. Ao final, o script imprime a URL pública da aplicação. Abra no navegador e teste.

O script `implantar-rm93624.sh` executa, de ponta a ponta:

1. Cria o Resource Group
2. Cria o recurso **Azure AI Vision** (Computer Vision)
3. Lê o **endpoint** e a **chave** do serviço
4. Cria o **App Service Plan** (Linux)
5. Cria o **Web App** (runtime Python 3.11)
6. Configura as variáveis de ambiente com as credenciais do Vision
7. Define o comando de inicialização (gunicorn)
8. Empacota e **publica a aplicação via Azure CLI** (`az webapp deploy`)

---

## 🔗 Como a aplicação se integra com a Visão Computacional

O arquivo [`app.py`](app.py) lê o **endpoint** e a **chave** do Azure AI Vision a
partir de variáveis de ambiente (`VISION_ENDPOINT` e `VISION_KEY`), definidas no
App Service pelo script de implantação — nenhuma credencial fica escrita no código.

Quando o operador envia uma imagem, a aplicação faz uma requisição **HTTP POST**
para o endpoint **Analyze Image (v3.2)** do Computer Vision, enviando os bytes da
imagem no corpo da requisição e solicitando três recursos visuais:

```
POST {VISION_ENDPOINT}/vision/v3.2/analyze?visualFeatures=Description,Tags,Objects
Header: Ocp-Apim-Subscription-Key: {VISION_KEY}
Body:   <bytes da imagem>  (Content-Type: application/octet-stream)
```

O serviço responde em **JSON**, de onde a aplicação extrai:

- `description.captions` → **descrição textual** gerada pela IA (requisito 4);
- `tags` → **etiquetas** reconhecidas na imagem (requisito 5);
- `objects` → **objetos** detectados, com sua localização (requisito 5).

Esses dados são então renderizados na página para apoiar o operador na análise do
ambiente monitorado pelo drone.

### Tratamento de erros (requisito 6)

A aplicação exibe mensagens adequadas para:

- imagem **inválida** ou formato não suportado;
- imagem **acima de 4 MB** (limite do serviço);
- **falha na análise** (resposta de erro do Azure AI Vision);
- **serviço indisponível** (falha de conexão / credenciais ausentes).

---

## 🧪 Teste local (opcional)

> O teste local serve apenas para desenvolvimento. A **entrega oficial deve rodar
> na nuvem** (implantação em localhost recebe nota zero).

```bash
pip install -r requirements.txt
export VISION_ENDPOINT="https://<seu-recurso>.cognitiveservices.azure.com/"
export VISION_KEY="<sua-chave>"
python app.py
# abra http://localhost:8000
```

---

## 🧹 Limpeza (após a entrega)

Para apagar todos os recursos e evitar custos:

```bash
az group delete --name rg-dronevision-rm93624 --yes --no-wait
```

---
