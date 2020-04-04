FROM node:current-slim

LABEL "com.github.actions.name"="Markdownlint Action"
LABEL "com.github.actions.description"="Tests all Markdownfiles"
LABEL "com.github.actions.icon"="mic"
LABEL "com.github.actions.color"="purple"

LABEL "repository"="https://github.com/DK-Hostmaster/DKHM-RFC-Delete-Domain"

RUN npm install -g markdownlint-cli

ADD entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
