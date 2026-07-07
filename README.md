# navemae-homelab

Configurações do meu servidor caseiro (`navemaehomelab`) — um notebook antigo rodando
[CasaOS](https://casaos.zimaspace.com/) sobre Ubuntu Server, com stack de mídia,
DNS local e ferramentas de acesso remoto.

## Hardware

| Item | Spec |
|---|---|
| CPU | Intel Core i5-5200U @ 2.20GHz (2c/4t) |
| RAM | 4 GB |
| Disco | 1 TB (ext4, disco único) |
| Rede | Ethernet (Realtek r8168) + Wi-Fi |

## Sistema base

- **Ubuntu Server 22.04 LTS** (kernel 5.15)
- **CasaOS v0.4.15** — painel e gerenciador dos apps Docker
- **Docker 26.0** + docker compose plugin
- **Tailscale** — acesso remoto via VPN mesh
- **Samba** — compartilhamento de arquivos na rede local (`/media/arquivos`)
- **Ollama** — servidor de LLMs local (instalado, mas **desativado** — 4 GB de RAM não rodam modelo útil)
- **rclone** — daemon rcd para sync com nuvem (config com credenciais fora do repo)

## Serviços (apps Docker via CasaOS)

| App | Imagem | Porta | Função |
|---|---|---|---|
| Jellyfin | `linuxserver/jellyfin:10.11.11` | 8097 | Servidor de mídia |
| Radarr | `linuxserver/radarr:5.17.2` | 7878 | Gerência de filmes |
| Sonarr | `linuxserver/sonarr:4.0.2` | 8989 | Gerência de séries |
| Bazarr | `linuxserver/bazarr:1.5.1` | 6767 | Legendas |
| Prowlarr | `linuxserver/prowlarr:1.14.3` | 9696 | Indexadores |
| qBittorrent | `hotio/qbittorrent:release-4.6.2` | 8181 | Cliente torrent |
| Ombi | `linuxserver/ombi:4.43.5` | 3579 | Pedidos de mídia |
| FlareSolverr | `flaresolverr/flaresolverr:v3.3.16` | 8191 | Proxy anti-Cloudflare p/ indexadores |
| Pi-hole | `pihole/pihole:2024.02.2` | 8800 / 53 | DNS + bloqueio de anúncios |
| Portainer | `portainer/portainer-ce:2.20.0-alpine` | 9443 | Gerência de containers |
| Chromium | `linuxserver/chromium` | 4000 | Navegador remoto (web) — mantido **parado** p/ economizar RAM |

Os dados dos apps ficam em `/DATA/AppData/<app>` e a mídia em `/DATA/Media`
(padrão do CasaOS).

## Estrutura do repo

```
apps/<app>/docker-compose.yml   # compose de cada app (sanitizado)
system/samba/smb.conf           # compartilhamento de rede
system/systemd/*.service        # units do Ollama e rclone
system/packages/apt-manual.txt  # pacotes apt instalados manualmente
system/zram/zramswap            # /etc/default/zramswap (swap comprimido)
system/sysctl.d/99-zram.conf    # swappiness ajustado para zram
system/docker/daemon.json       # rotação de logs do Docker
```



## Como reproduzir

1. Instale Ubuntu Server 22.04 e o CasaOS (`curl -fsSL https://get.casaos.io | sudo bash`).
2. Instale os pacotes base: `xargs -a system/packages/apt-manual.txt sudo apt install -y`
   (revise a lista antes — alguns pacotes como `r8168-dkms` são específicos deste hardware).
3. Para cada app: no CasaOS, use **Install a customized app → Import** e cole o
   `docker-compose.yml` correspondente, ou rode direto com
   `docker compose up -d` dentro da pasta do app.
   Ajuste caminhos de volume e **troque os placeholders de senha**.
4. Copie `system/samba/smb.conf` para `/etc/samba/` e os units de
   `system/systemd/` para `/etc/systemd/system/`, depois
   `sudo systemctl daemon-reload && sudo systemctl enable --now smbd ollama`.
5. Tailscale e rclone exigem autenticação própria (`tailscale up`, `rclone config`).

