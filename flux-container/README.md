# Dev container

## Что где хранится

| Что | Где | Переживает |
|---|---|---|
| Сессии и настройки Claude (`~/.claude`, включая `.claude.json`) | на маке, `~/.devcontainer-state/arch/claude` | удаление контейнера и томов Docker |
| Навыки `~/.agents` (ru-text) | на маке, `~/.devcontainer-state/arch/agents` | то же |
| История bash и zsh | на маке, `~/.devcontainer-state/arch/shell-history` | то же |
| VS Code Server и расширения | том Docker `arch-vscode-server` | Rebuild; при потере скачаются заново |
| Утилиты (flux, kustomize, prek и другие) | образ, версии в `Dockerfile` | кешируются слоями Docker |

Папки на маке создаёт `initializeCommand` перед стартом контейнера.

## Если что-то пошло не так

- **Сборка образа упала.** VS Code предложит открыть контейнер в режиме восстановления. Данные в `~/.devcontainer-state` не пострадают. Чаще всего причина — сеть до GitHub при скачивании утилит.
- **Старт висит на «Downloading VS Code Server».** Это сеть до `update.code.visualstudio.com`, а не конфиг: проверьте прокси в Docker Desktop. После первой успешной загрузки сервер берётся из тома `arch-vscode-server`.
- **Обновить утилиту.** Поменять `ARG` в `Dockerfile` и выполнить Rebuild. Версии flux, flux-operator, kustomize, kubeconform и yq держите такими же, как в `.gitlab-ci.yml`.
