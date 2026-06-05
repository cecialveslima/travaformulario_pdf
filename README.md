# PDFFlatten

Utilitário Java para achatar campos de formulário PDF e garantir compatibilidade com visualizadores mobile (WhatsApp iOS/Android).

---

## Problema resolvido

O webapp `interfaces` usa GeneXus 16 EV8, cujo `gxclassR.jar` ao preencher campos PDF via `SetFields` grava a fonte **Helvetica 14pt** no AP stream, ignorando a fonte original do template (**Times Roman 10pt**). Além disso, a fonte `/TiRo` (alias Times Roman) não fica embutida no PDF, causando campos em branco em visualizadores mobile.

Esta classe resolve os dois problemas em uma única chamada, sem alteração no código GeneXus.

---

## Dependências (Build Path no Eclipse)

| JAR | Caminho no servidor |
|-----|-------------------|
| `itext-2.1.7.jar` | `WEB-INF/lib/itext-2.1.7.jar` |
| `pdfbox-app-2.0.24.jar` | `WEB-INF/lib/pdfbox-app-2.0.24.jar` |

> **Atenção:** NÃO usar `PDFBox-0.7.3.jar` — é a versão antiga (2006) e não possui as classes necessárias. Usar obrigatoriamente o `pdfbox-app-2.0.24.jar`.

---

## Como compilar

### Pelo Eclipse
1. Clique com botão direito no projeto → **Build Path → Configure Build Path → Libraries → Add External JARs**
2. Adiciona `itext-2.1.7.jar` e `pdfbox-app-2.0.24.jar`
3. Clique com botão direito no projeto → **Export → Java → JAR file**
4. Seleciona o projeto, define o caminho de saída (`PdfFlatten.jar`) → **Finish**

### Pela linha de comando
```bash
javac -cp itext-2.1.7.jar:pdfbox-app-2.0.24.jar \
      -d bin \
      src/com/interfaces/PDFFlatten.java \
      src/com/interfaces/Main.java

jar cf PdfFlatten.jar -C bin .
```
> No Windows substitua `:` por `;` no classpath.

---

## Deploy

Copiar `PdfFlatten.jar` para:
```
/usr/local/apache-tomcat-9.0.89/webapps/interfaces/WEB-INF/lib/
```

Reiniciar o Tomcat após substituição:
```bash
systemctl restart tomcat
# ou
/usr/local/apache-tomcat-9.0.89/bin/shutdown.sh && \
/usr/local/apache-tomcat-9.0.89/bin/startup.sh
```

---

## Uso

### GeneXus (sem alteração necessária)
```genexus
&PdfReadOnlyPDFFlatten.flattenFields(&TargetPDFPath, &xTargetPDFPath)
```

### Java direto / teste
```bash
java -cp PdfFlatten.jar:itext-2.1.7.jar:pdfbox-app-2.0.24.jar \
     com.interfaces.Main entrada.pdf saida.pdf
```

---

## O que o flattenFields faz internamente

**Passo 1 — iText (correção de fonte)**
- Percorre todos os campos do formulário
- Substitui qualquer fonte no DA por `/TiRo 10 Tf` (Times Roman 10pt)
- Reescreve o AP stream de cada campo com a fonte correta
- Achata os campos (`setFormFlattening(true)`)

**Passo 2 — PDFBox (rasterização para mobile)**
- Renderiza o PDF achatado como imagem a **200 DPI**
- Reempacota as imagens em um novo PDF
- Elimina qualquer dependência de fontes externas
- Garante abertura correta em WhatsApp iOS/Android, Redmi e qualquer visualizador

---

## Constantes configuráveis

No topo da classe `PDFFlatten.java`:

```java
private static final String CORRECT_FONT = "/TiRo";  // alias da fonte no template
private static final String CORRECT_SIZE = "10";      // tamanho em pt
private static final int    MOBILE_DPI   = 200;       // 150=menor arquivo, 300=alta qualidade
```

---

## Contexto técnico

| Item | Detalhe |
|------|---------|
| Template PDF | `/TiRo 10 Tf` (alias Times Roman Type1) |
| Problema EV8 | `gxclassR.jar` de fev/2020 grava `Helv 14 Tf` |
| Solução EV9 | `gxclassR.jar` de abr/2020 preserva fonte original |
| Workaround | Correção no AP stream + rasterização PDFBox |
| Servidor | Tomcat 9.0.89, Java, Linux |
| Webapp com problema | `interfaces` (GeneXus 16 EV8) |
| Webapp sem problema | `webcards4` (GeneXus 16 EV9) |
