# Auditoria de Calçadas com Visão Computacional

Projeto desenvolvido para o checkpoint integrado **CPZÃO**, com o objetivo de aplicar Visão Computacional à auditoria de calçadas urbanas por meio de detecção de objetos usando YOLO e execução do modelo em um aplicativo móvel.

> **Importante:** revise os campos marcados como `[PREENCHER]` e substitua-os pelos dados reais do grupo, do dataset e do treinamento antes de entregar. Não informe métricas ou quantidades que não tenham sido verificadas.

## 1. Integrantes

| Nome completo | RM |
|---|---|
| Pedro Barros | RM97937 |
| [PREENCHER] | [PREENCHER] |
| Victor Kenzo | RM551649 |

## 2. Contexto e objetivo

A qualidade das calçadas e a acessibilidade urbana são desafios importantes nas cidades. Calçadas podem apresentar defeitos, obstáculos ou elementos de acessibilidade que afetam a circulação dos pedestres.

Este projeto desenvolve um pipeline de Visão Computacional para detectar objetos relevantes à auditoria de calçadas. O fluxo contempla:

1. Aquisição e curadoria das imagens.
2. Anotação dos objetos de interesse.
3. Organização dos dados em conjuntos de treino, validação e teste.
4. Treinamento e avaliação de um modelo YOLO.
5. Exportação do modelo treinado para ONNX.
6. Integração do modelo ao aplicativo `yoloinferApp` para inferência no smartphone.

## 3. Classes detectadas

O modelo foi treinado para reconhecer as seguintes classes, nesta ordem:

| ID | Classe | Descrição |
|---:|---|---|
| 0 | `lixo` | Resíduos ou lixo presentes na calçada. |
| 1 | `piso_tatil` | Piso tátil utilizado para orientação e acessibilidade. |
| 2 | `poste` | Postes que podem estar presentes na área de circulação. |

A ordem das classes deve ser mantida de forma consistente entre as anotações, o arquivo `data.yaml`, o treinamento e a aplicação. Os nomes exibidos nas caixas delimitadoras do aplicativo foram ajustados para corresponder às classes do modelo.

## 4. Dataset e curadoria

As imagens devem representar situações variadas de calçadas, condições de iluminação, perspectivas, distâncias e ocorrências dos objetos de interesse. As caixas delimitadoras devem envolver os objetos de maneira consistente.

### Estatísticas do dataset

| Item | Quantidade |
|---|---:|
| Total de imagens rotuladas | 159 |
| Imagens de treino | 111 |
| Imagens de validação | 48 |
| Número de classes | 3 |
| Ferramenta de anotação | Roboflow |

**Requisito da atividade:** utilizar pelo menos 60 imagens rotuladas e pelo menos 3 classes distintas. Registre aqui a contagem efetivamente conferida, sem considerar imagens sem anotação válida como imagens rotuladas.

### Estrutura esperada

```text
dataset/
├── data.yaml
├── train/
│   ├── images/
│   └── labels/
└── val/
    ├── images/
    └── labels/
```

O `data.yaml` deve apontar para os caminhos corretos e declarar as classes na mesma ordem apresentada na tabela anterior. O formato de rótulos YOLO utiliza, normalmente, uma linha por objeto com o ID da classe e as coordenadas normalizadas da caixa delimitadora.

## 5. Treinamento do modelo

Foi utilizado um modelo da família YOLO para detecção de objetos. O arquivo de pesos treinados (`.pt`) foi exportado para ONNX para ser utilizado no aplicativo.

| Configuração | Valor |
|---|---|
| Arquitetura/modelo | YOLO11n |
| Épocas de treinamento | 500 |
| Tamanho da imagem no treinamento | 640 |
| Melhor arquivo de pesos | `best.pt` |

### Como reproduzir o treinamento

Instale as dependências utilizadas pelo projeto:

```bash
pip install ultralytics
```

Execute o notebook de treinamento através do link abaixo, ou o script equivalente utilizado no projeto. Os caminhos do dataset e os parâmetros devem estar configurados de acordo com o ambiente.

> https://colab.research.google.com/drive/1pE02WUSmt2LkXZgapcrtvYOeq2B8HKQZ?usp=sharing

## 6. Avaliação e métricas

| Métrica | Resultado |
|---|---:|
| mAP@50 | 0.553 |
| mAP@50–95 | 0.303 |
| Época com melhor resultado | 241 |


## 7. Exportação para ONNX

O aplicativo base `yoloinferApp` utilizado neste projeto espera uma entrada de imagem com dimensões **`[1, 3, 192, 192]`**. Por isso, o modelo ONNX utilizado no aplicativo foi exportado com tamanho de entrada 192 × 192.

Exemplo de exportação com Ultralytics:

```python
from ultralytics import YOLO

model = YOLO("caminho/para/best.pt")

model.export(
    format="onnx",
    imgsz=192,
    dynamic=False
)
```

Substitua o caminho pelo local real do arquivo `best.pt`. O ONNX resultante normalmente é criado próximo ao arquivo de pesos de origem.

**Observação:** a resolução de 192 × 192 é uma exigência de compatibilidade com a versão do aplicativo utilizada. Uma resolução menor pode reduzir a precisão em objetos pequenos. Após exportar, valide o arquivo no aplicativo e teste se as detecções e as caixas delimitadoras estão corretas.

## 8. Aplicativo móvel (`yoloinferApp`)

O deploy utiliza como base o repositório [Mjrovai/yoloinferApp](https://github.com/Mjrovai/yoloinferApp). O código foi adaptado para utilizar o modelo ONNX treinado para auditoria de calçadas.

### Alteração dos nomes das classes

Na aplicação, os nomes das classes devem corresponder à ordem usada durante o treinamento. No arquivo JavaScript responsável pela interface, a lista de classes foi ajustada para:

```javascript
const CLASS_NAMES = [
    'lixo',
    'piso_tatil',
    'poste'
];
```

Se as classes ou sua ordem forem alteradas no treinamento, atualize também essa lista. Caso contrário, o aplicativo poderá mostrar nomes incorretos nas caixas delimitadoras.

### Como executar

A aplicação é uma **Progressive Web App (PWA)** feita com JavaScript e Vite. A inferência acontece no navegador com ONNX Runtime Web (WASM); o modelo é executado no próprio dispositivo.

**Pré-requisitos**
- Node.js e npm instalados.
- O arquivo do modelo exportado para ONNX com entrada `[1, 3, 192, 192]`.

1. Clone este repositório, abra um terminal na pasta raiz do aplicativo, onde está o `package.json`.
2. Instale as dependências:

   ```bash
   npm install
   ```

3. Inicie o servidor de desenvolvimento com o script definido no projeto:

   ```bash
   npm run dev
   ```

4. Use o endereço HTTPS apresentado no terminal. Para testar em um celular na mesma rede Wi-Fi, abra no navegador do celular o endereço `Network` apresentado pelo Vite, normalmente semelhante a `https://192.168.1.42:5173`. O endereço exato depende da rede e será mostrado no terminal.
5. Se o navegador mostrar um aviso de certificado durante o desenvolvimento HTTPS local, aceite-o para abrir a aplicação. O acesso à câmera requer HTTPS; em alguns ambientes móveis, `localhost` ou HTTP não permitem usar a câmera.
6. Para testar no computador, abra o endereço local indicado pelo Vite no navegador.


### Como usar o aplicativo

1. Abra a aplicação no navegador e clique em **Model**.
2. Selecione o arquivo `.onnx` treinado para a auditoria de calçadas. No computador, o navegador compatível abre um seletor de arquivos filtrado para ONNX. No iPhone/iPad, se aparecer um menu com opções como biblioteca de fotos, câmera e arquivos, escolha **Files/Arquivos** e localize o modelo.
3. Aguarde o carregamento do modelo. A versão usada neste projeto espera entrada de imagem de **192 × 192 pixels**. Se o modelo tiver sido exportado com outra resolução, a inferência poderá falhar.
4. Para testar uma imagem estática, clique em **Upload** e escolha uma imagem. Confira se as caixas delimitadoras e os nomes das classes estão corretos.
5. Para inferência ao vivo, clique em **Start** e permita o acesso à câmera quando o navegador solicitar. Use a câmera traseira do smartphone para a demonstração.
6. Ajuste os controles de confiança (**Confidence**) e IoU, se necessário, para observar como eles afetam as detecções. Use **Stop** para interromper a câmera.

### Atualizar a versão durante o desenvolvimento

A aplicação inclui um Service Worker que pode manter arquivos em cache. Se alterações no código ou no modelo não aparecerem, faça uma atualização forçada (`Ctrl + F5` no Windows) ou abra a aplicação em uma janela anônima. No celular, tente recarregar a página ou abri-la em uma sessão privada.

### Build e publicação (opcional)

Para gerar a versão de produção:

```bash
npm run build
```

O Vite gera os arquivos finais na pasta `dist/`. Para publicar em um serviço como Netlify, configure a publicação para usar `dist/` como diretório de saída. A hospedagem deve fornecer HTTPS para que o acesso à câmera funcione em dispositivos móveis.

## 9. Limitações e próximos passos

- A qualidade das detecções depende da variedade, quantidade e consistência das imagens anotadas.
- Objetos pequenos podem perder detalhes quando a entrada é reduzida para 192 × 192.
- A inferência no celular pode variar conforme o aparelho, o navegador e as condições de iluminação.
- A detecção de um objeto não substitui uma avaliação técnica completa da acessibilidade ou das condições da calçada.

## 10. Referências

- **Auditoria de Calçadas — Projeto e Metodologia:** https://falzoni.com.br/auditoriacalcadas/
- **Notebook de aquisição e preparação de dados (Google Colab):** https://colab.research.google.com/drive/1WwkI80sm8gJfrYIiCIOiysChPeg1Km_B?usp=sharing
- **Aplicativo base `yoloinferApp`:** https://github.com/Mjrovai/yoloinferApp
- **Documentação de exportação do Ultralytics YOLO:** https://docs.ultralytics.com/modes/export/
