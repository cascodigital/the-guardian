# The Guardian — Auditoria de Email

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![License](https://img.shields.io/badge/License-MIT-blue)
![Author](https://img.shields.io/badge/Author-Casco%20Digital-orange)

![Cloudflare](https://img.shields.io/badge/Cloudflare-Workers-F38020?style=flat-square&logo=cloudflare&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-CSS3-E34F26?style=flat-square&logo=html5&logoColor=white)

Ferramenta publica de auditoria de seguranca de email. Analisa registros **SPF**, **DMARC** e **MX** de qualquer dominio e gera um Score de Risco (0-100) em segundos, identificando vulnerabilidades de spoofing.

**Live:** [cascodigital.com.br/guardian](https://cascodigital.com.br/guardian)

## Como funciona

1. Usuario informa o dominio
2. Cloudflare Worker consulta os registros DNS via Cloudflare DNS-over-HTTPS
3. SPF, DMARC e MX sao analisados e pontuados
4. Frontend exibe o Score de Risco com recomendacoes de correcao

## Estrutura

```
the-guardian/
├── guardian.html           # Interface completa (single-file)
├── functions/api/
│   └── audit.js            # Worker: consulta DNS e calcula score
└── README.md
```

## Deploy

### Cloudflare Pages

1. Conecte este repositorio em **Cloudflare Pages** > **Create a project** > **Connect to Git**
2. Configuracao de build:
   - Framework preset: **None**
   - Build command: (vazio)
   - Build output directory: `/`

Sem variaveis de ambiente necessarias — usa exclusivamente a API publica de DNS do Cloudflare.

## Stack

- HTML/CSS/JS vanilla (sem dependencias)
- Cloudflare Workers (backend serverless)
- Cloudflare DNS-over-HTTPS (`cloudflare-dns.com/dns-query`)

---

Desenvolvido com 🐢 (e cafe) por **Casco Digital**.
