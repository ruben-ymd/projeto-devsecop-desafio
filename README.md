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
PASSO A IMPLEMENTAR — Secrets Scanning com Gitleaks
    - Ferramenta para escanear dados sensíveis no código. 
    - Foi identificado Chaves/Tokens/ID mas a ferramenta não identificou devido aos dados serem um exemplo (segundo IA).
    - Inseri uma chave dado pelo IA como exemplo e a ferramenta identificou.
    - Apesar de ter colocado somente para comentário, a ferramenta acusou.
    - Retiramos os códigos e a ferramena passou sem detectar outro incidente.

PASSO A IMPLEMENTAR — SAST com Semgrep
    - Objetivo da ferramente é buscar no código fonte falhas de segurança, práticas ruins de programação e vulnerabilidade lógicas. 
    - Instalei o semgrep via pip
    - Configurei o scan
    - Tive alguns problemas de identação.
    - Foi identificado um hardcoded secret no script.js - risco de injeção de código 
    - se alguém inserisse comandos maliciosos em vez de uma tarefa comum, o sistema executaria esse código arbitrariamente no navegador de quem estivesse a aceder.

PASSO A IMPLEMENTAR — SCA com Grype
    - Instalei o package lock - arquivo de controle gerado automaticamente pelo gestor de pacotes do Node.js (npm) sempre que dependências são instaladas num projeto.
    - Instalado Grype
    - Foram identificados na ferramenta as dependências desatualizadas.
    - Atualizamos mas mesmo assim a ferramenta ainda acusa dependências desatualizadas.
    - Instalei o node para verificar as versões mais recentes os package.
    - Rodei a Grype e achou vulnerabilidade em pipeline.yml em linha 133 actions/downlaod-artifact. versao instalada v4
    - Rodei novamente e passou com sucesso.


## URL de Produção
> Adicione aqui o link do GitHub Pages após o deploy.
https://ruben-ymd.github.io/projeto-devsecop-desafio/