# Fluxo de sonorização (SFX) — resumo e uso nos vídeos de economia

> Memória de trabalho do fluxo de sound design. Os scripts citados aqui
> (`look.py`, `from_cues.py`, `mix.py` etc.) **não estão neste repositório**;
> este arquivo registra o fluxo para quando o kit estiver disponível.

## 1. Pipeline

```
look.py (ver o filme)
  -> from_cues.py        lista de eventos do projeto, ou chamadas do script que sintetiza o som (audio.cjs)
  -> ui_events.py        gravação de tela, região por região
  -> from_hyperframes.py timeline com data-start (HyperFrames)
  -> find_moments.py     análise de quadros -> suggest_cues.py
        -> (você + Claude editam a lista de eventos)
        -> mix.py
        -> verify.py, review.py, critique.py   (repetir até nota 8+)
```

## 2. Comandos de referência

```bash
# 1. projeto com lista de eventos (docs/cues.json, scripts/cues.json) OU script que sintetiza o som (audio.cjs)
python scripts/from_cues.py PROJETO|docs/cues.json --video RENDER.mp4 --out work/cues.json
# 2. gravação de tela sem lista: ui_events.py ... (references/screencast.md), depois from_cues.py work/ui/events.json
# 3. só imagem:
find_moments.py FILME.mp4 --out work/moments
suggest_cues.py work/moments/moments.json --out work/cues.json
# 3b. animação suave/colorida sem lista (prévia de UI, ilustração, motion graphics):
#     a automação só vê 5–6 de ~40 movimentos. Usar:
find_moments.py FILME --out work/moments --fine
look.py FILME --at T --span 2.4 --step 0.1 --out work/at.png
#     escrever a PRÓPRIA lista de eventos a partir dos quadros (references/typed-cues.md)
from_cues.py - --ui library

# mixagem
python scripts/mix.py work/cues.json --out result/film_sfx.mp4 --plan   # o que vai onde; depois sem --plan
python scripts/critique.py result/film_sfx_resolved_cues.json result/film_sfx_sfx_track.wav
python scripts/levels.py   result/film_sfx_resolved_cues.json result/film_sfx_sfx_track.wav --json work/edits.json

# filme cujo som é SÓ SFX (sem música/voz própria):
python scripts/texture.py work/cues.json --moments work/moments/moments.json  # sons pequenos nos buracos
python scripts/bed.py work/cues.json [--list]                                 # "bed": gravação longa e escura por baixo
#   (rodar mix.py de novo depois de cada um)

# comparação com 9 reels de SFX finalizados (só números, não muda o veredito)
python scripts/critique.py RESOLVED.json WAV --style clean-sfx-refs

# escuta humana barata: página com as perguntas da crítica e veredito em um toque
python scripts/listen_page.py build work/listen --moments result/film_sfx.mp4 --critique RESOLVED.json WAV
python scripts/listen_page.py serve work/listen [--lan]
python scripts/listen_page.py results work/listen --log
# modelo de mascaramento (quais sons os outros escondem; precisa do venv)
python scripts/mix.py work/cues.json --out result/film_sfx.mp4 --masking
```

## 3. Como entra nos vídeos de economia (skill `video-economia`)

Encaixa na **Etapa 2 — Edição, item 5** ("efeitos sonoros pontuais nos cards"),
depois que os cards estão sincronizados e antes da exportação 9:16.

1. **Fonte da lista de eventos:** os cards são feitos em HyperFrames/Remotion,
   então a timeline já existe → usar `from_hyperframes.py` (timeline `data-start`)
   ou `from_cues.py` com o projeto. Não depender de `find_moments.py`.
2. **Mapa card → som** (pontual, curto, nunca sobre a fala):

   | Card | Som sugerido |
   |------|--------------|
   | Card de dado (número grande) | "pop"/impacto curto na entrada do número |
   | Lower-third de pensador | "whoosh" suave na entrada |
   | Card de definição | clique/tique leve |
   | Gráfico animado | sobe/desce acompanhando a linha ou as barras; tique no valor final |
   | Gancho (0–3s) | um único acento forte |

3. **O vídeo tem voz e música** → **não** usar `texture.py` nem `bed.py`
   (são só para filmes cujo som é apenas SFX).
4. **Níveis:** SFX abaixo da voz; música baixa. Rodar `levels.py` e o
   `mix.py --masking` para confirmar que nenhum efeito cobre a fala.
5. **Loop de qualidade:** `verify.py` → `review.py` → `critique.py` até nota **8+**;
   se der, uma rodada no `listen_page.py` pelo celular.
6. **Relatório final (Etapa 3):** incluir a lista de efeitos usados (tempo, card,
   som) e a nota final da crítica.
