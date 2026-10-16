# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech.
Você receberá este projeto com vulnerabilidades propositais e uma pipeline incompleta.
Seu objetivo é **implementar a pipeline de segurança** e **corrigir as vulnerabilidades**.

## Estado atual
A pipeline está **incompleta**. Os steps de segurança precisam ser implementados por você.

## Sua missão
1. Implementar os steps de segurança no `pipeline.yml`
2. Fazer a pipeline **quebrar** ao detectar os problemas
3. Corrigir as vulnerabilidades encontradas
4. Fazer a pipeline **passar** com tudo verde ✅
5. Documentar o funcionamento da pipeline neste README

## O que implementar
- [ ] Secrets Scanning com **Gitleaks**
- [ ] SAST com **Semgrep**
- [ ] SCA com **Grype**
- [ ] Assinatura do artefato com cosign
- [ ] Deploy com **GitHub Pages**

## Como a pipeline funciona
> **Substitua este bloco pela sua explicação após implementar a pipeline.**
> Descreva cada step, o que ele faz e por que ele é importante para a segurança.

# Desafio DevSecOps — Gerenciador de Tarefas


## O que é uma pipeline?

Uma pipeline é uma sequência automatizada de tarefas executadas quando uma alteração é enviada ao repositório. Neste projeto, ela confere o código e as dependências, prepara e assina os arquivos do site e, se as etapas anteriores terminarem com sucesso, publica o site no GitHub Pages.

Na prática, essas verificações ajudam a encontrar problemas antes da publicação. Elas reduzem riscos, mas não garantem sozinhas que a aplicação esteja livre de falhas.

## Quando a pipeline é executada?

A pipeline em [pipeline.yml](.github/workflows/pipeline.yml) é iniciada quando há um `push` para a branch `main`. Ela tem dois jobs:

1. **Build, segurança e assinatura:** analisa e prepara os arquivos do site.
2. **Verificação e deploy:** recebe os arquivos preparados e publica o site. Este job só começa se o primeiro terminar com sucesso.

## Job 1 — Build, segurança e assinatura

As etapas são executadas na ordem abaixo:

### 1. Baixar o código (Checkout)

O GitHub Actions copia o conteúdo do repositório para o ambiente temporário onde a pipeline será executada. Sem isso, as ferramentas não teriam os arquivos para analisar ou publicar.

### 2. Inserir o segredo no JavaScript

A etapa lê `MINHA_SENHA_PROJETO` dos segredos configurados no GitHub e substitui o marcador correspondente em `src/script.js`.

**Atenção:** guardar um valor em GitHub Secrets evita colocá-lo diretamente no repositório, mas não o torna seguro para inserir em JavaScript enviado ao navegador. Os arquivos do site são públicos; quem acessa a página pode inspecionar o JavaScript e obter o valor inserido. Segredos reais não devem ser enviados ao navegador. Eles devem permanecer em um serviço de backend. Se um segredo real já foi publicado, ele deve ser revogado e substituído.

### 3. Build

Atualmente, esta etapa lista os arquivos de `src/` e imprime `Build OK!`. Ela serve como uma etapa demonstrativa, mas **não compila nem testa a aplicação**.

### 4. Procurar segredos com Gitleaks (Secrets Scanning)

O Gitleaks procura padrões que possam indicar senhas, tokens ou chaves expostos no código e no histórico analisado. Isso é importante porque uma credencial publicada pode permitir acesso indevido a serviços ou dados.

Durante o desafio, um valor de exemplo foi detectado mesmo estando em um comentário. Isso mostra que comentários também podem ser analisados. A ferramenta pode não reconhecer todo valor fictício como um segredo; nunca use credenciais reais para testar.

### 5. Analisar o código com Semgrep (SAST)

SAST significa análise estática de segurança: a ferramenta examina o código sem executar a aplicação. O Semgrep usa regras para procurar padrões associados a falhas e práticas inseguras. A configuração usada combina regras automáticas com regras para XSS e faz a etapa falhar se encontrar problemas (`--error`).

No exercício, a análise apontou um segredo codificado diretamente no JavaScript e um risco de injeção de código ao exibir uma tarefa informada pelo usuário. O uso de `eval` foi retirado, mas inserir conteúdo do usuário com `innerHTML` ainda pode permitir XSS. Para texto simples, prefira `textContent` ou `innerText` e valide a correção com a análise.

### 6. Analisar dependências com Grype (SCA)

SCA significa análise de composição de software: em vez de procurar falhas na lógica escrita pela equipe, ela verifica os pacotes de terceiros usados pelo projeto e compara versões com vulnerabilidades conhecidas.

A pipeline gera um `package-lock.json`, instala o Grype e analisa os arquivos do projeto com `grype dir:. --fail-on medium`. Assim, vulnerabilidades encontradas com severidade média ou superior fazem a etapa falhar. O arquivo de lock registra as versões resolvidas das dependências e ajuda a tornar a análise mais precisa e reproduzível. Atualizar um pacote pode exigir nova análise; uma dependência desatualizada nem sempre significa que há uma vulnerabilidade conhecida, e uma análise sem alertas não prova que tudo está seguro.

### 7. Empacotar o site

Os arquivos de `src/` são reunidos em `artifact.tar`. Esse pacote é o artefato que será transferido para o job de deploy, evitando que o segundo job precise reconstruir o site.

### 8. Calcular o hash SHA-256

A pipeline calcula um resumo do pacote e o salva em `artifact.tar.sha256`. Um hash funciona como uma impressão digital: se o conteúdo mudar, o resumo também muda. O hash é enviado junto com o artefato, mas atualmente o job de deploy **não o compara** antes da publicação.

### 9. Assinar o artefato com Cosign

O Cosign cria uma assinatura para `artifact.tar`, usando a identidade temporária do GitHub Actions (assinatura sem chave local, também chamada de *keyless*). A assinatura ajuda a demonstrar a origem do pacote e a detectar alterações quando ela é verificada.

### 10. Enviar o pacote para o job de deploy

O artefato, a assinatura e o hash são enviados como `site-assinado` para que o próximo job possa baixá-los.

## Job 2 — Verificação e deploy

Este job declara `needs: build`, portanto só é executado depois do sucesso do job anterior.

### 1. Baixar o artefato e instalar o Cosign

O job baixa os arquivos `site-assinado` e instala o Cosign para poder trabalhar com a assinatura.

### 2. Verificar a assinatura (ainda desativado)

O comando que verificaria a assinatura com `cosign verify-blob` está comentado no workflow. Portanto, **apesar de a assinatura ser criada, ela não é verificada antes do deploy**. A etapa de extração e publicação não deve ser considerada protegida por essa verificação enquanto o comando continuar desativado.

### 3. Extrair e publicar no GitHub Pages

O pacote é extraído para a pasta `site/`, preparado para o GitHub Pages e publicado. Essa etapa torna o site acessível pela internet.

## O que foi praticado

- **Gitleaks:** testar a busca por possíveis segredos e remover valores sensíveis do código.
- **Semgrep:** configurar análise estática e investigar padrões de código que podem permitir injeção.
- **Grype:** gerar o arquivo de lock, analisar dependências e revisar alertas de vulnerabilidades conhecidas.
- **Cosign:** assinar o artefato para registrar sua origem e integridade.
- **GitHub Pages:** publicar o site depois das etapas do job de build.

## URL de Produção
> Adicione aqui o link do GitHub Pages após o deploy. 

https://ruben-ymd.github.io/projeto-devsecop-desafio/