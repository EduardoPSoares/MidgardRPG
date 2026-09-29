# MidgardRPG

![Java](https://img.shields.io/badge/Java-21-E76F00?logo=openjdk&logoColor=white)
![Paper](https://img.shields.io/badge/Paper-1.21-2F80ED)
![Redis](https://img.shields.io/badge/Redis-pub%2Fsub-DC382D?logo=redis&logoColor=white)
![License](https://img.shields.io/github/license/EduardoPSoares/MidgardRPG)

Plugin de RPG modular para servidores Minecraft (Paper 1.21), escrito em Java. Um núcleo (`midgard-core`) oferece
serviços comuns e cada sistema de jogo é um módulo independente, carregado pelo `midgard-loader`.

## Destaques

- **Arquitetura modular**: 13 módulos de jogo (personagem, classes, combate, comandos, economia, essentials,
  itens, profissões, raças, magias, segurança, desempenho e integração com MythicMobs) sobre um núcleo compartilhado.
- **Núcleo**: atributos e modificadores, efeitos de status, sistema de dano próprio, perfis de jogador
  persistentes, loot tables, leaderboards, scoreboard, GUIs paginadas, i18n com validação de chaves ausentes.
- **Persistência e rede**: MySQL (HikariCP) e sincronização entre servidores via Redis; módulo `midgard-proxy`
  para Velocity.
- **Compatibilidade de versão**: a camada `midgard-nms` isola o código que depende da versão do servidor.
- **Integrações**: Vault, WorldGuard, MythicMobs, PlaceholderAPI, TAB.

## Stack

Java 21 · Maven (multi-módulo) · Paper API · MySQL/HikariCP · Redis · Velocity

## Estrutura

```
midgard-core/      serviços compartilhados (atributos, combate, perfis, GUI, i18n, banco, Redis)
midgard-modules/   um módulo por sistema de jogo
midgard-loader/    plugin que carrega o núcleo e os módulos
midgard-nms/       código específico da versão do servidor
midgard-proxy/     plugin Velocity
```

## Arquitetura

```mermaid
flowchart LR
    subgraph Servidor Paper
        loader[midgard-loader] --> core[midgard-core]
        loader --> mods[midgard-modules<br/>combate, classes, magias,<br/>economia, profissões...]
        mods --> core
        core --> nms[midgard-nms<br/>v1_21]
    end
    core <-->|perfis| mysql[(MySQL)]
    core <-->|pub/sub| redis[(Redis)]
    proxy[midgard-proxy<br/>Velocity] <-->|pub/sub| redis
```

### Troca de servidor sem perder dados

Antes de mover o jogador, o proxy ([MidgardProxy](https://github.com/EduardoPSoares/MidgardProxy)) pede ao
servidor de origem que salve o perfil e espera a confirmação (até 2 s) antes de conectar no destino.

```mermaid
sequenceDiagram
    participant P as MidgardProxy (Velocity)
    participant R as Redis
    participant A as Servidor de origem
    participant B as Servidor de destino
    P->>R: publish midgard:sync:req_save
    R->>A: req_save
    A->>A: salva o perfil no MySQL
    A->>R: publish midgard:sync:saved
    R->>P: saved
    P->>B: conecta o jogador
    B->>B: carrega o perfil atualizado
```

## Como compilar

```bash
./mvnw package -pl midgard-loader -am
```

O jar sai em `midgard-loader/target/`.
