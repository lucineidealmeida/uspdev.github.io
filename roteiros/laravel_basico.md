---
layout: default
title: Roteiro - Laravel Básico
permalink: /roteiros/laravel_basico
---

Scripts usados na criação do ambiente do curso de laravel básico, para rodá-los,
exporte seu token para usar a API do github, como por exemplo:

    export token='hdekjhdkjehfkje'

Dica para verificação dos limites de requests:

    curl  -i https://api.github.com/users/thiagogomesverissimo | grep RateLimit

1. Criar um arquivo com lista dos nomes dos repositórios que serão criados, exemplo:

        paulo
        maria
        pedro
        vanessa

2. Script para criação repositórios zerados no github:

        #!/bin/bash
        # run as: ./script1.sh lista.txt

        export prefix="disciplinas_"
        export organization="uspdev-labs"

        while IFS='' read -r line || [[ -n "$line" ]]; do
            curl -H "Authorization: token $token" https://api.github.com/orgs/$organization/repos -d '{"name":"'$prefix$line'"}'
        done < "$1"

2. Subir repositório base para o curso

        #!/bin/bash
        # run as: ./script2.sh lista.txt

        export prefix="disciplinas_"
        export organization="uspdev-labs"
        export base_repo="https://github.com/thiagogomesverissimo/disciplinas_start.git"
        export dest='/tmp/base_repo'

        rm -rf $dest
        git clone $base_repo $dest
        while IFS='' read -r line || [[ -n "$line" ]]; do
            cd $dest
            rm -rf .git
            git init
            git add .
            git commit -m 'Initial Commit'
            git remote add origin git@github.com:$organization/$prefix$line.git
            git push origin master
        done < "$1"

3. Criação issues

        #!/bin/bash
        # run as: ./script3.sh lista.txt

        prefix="disciplinas_"
        organization="uspdev-labs"
        issues="https://raw.githubusercontent.com/thiagogomesverissimo/disciplinas_start/master/issues.txt"

        curl -s $issues -o /tmp/issues.txt
        while IFS='' read -r line || [[ -n "$line" ]]; do
          while IFS='' read -r issue || [[ -n "$issue" ]]; do
            curl -H "Authorization: token $token" https://api.github.com/repos/$organization/$prefix$line/issues -d '{"title": "'"$issue"'"}'
          done < "/tmp/issues.txt"
          sleep 900 # github limit
        done < "$1"

4. Contar issues para conferência

        #!/bin/bash
        # run as: ./script3.sh lista.txt

        prefix="disciplinas_"
        organization="uspdev-labs"

        echo "repo: qtde issues"
        while IFS='' read -r line || [[ -n "$line" ]]; do
            test=$(curl -s -H "Authorization: token $token" https://api.github.com/repos/$organization/$prefix$line/issues | grep html_url | grep issues | wc -l)
            echo $prefix$line: $test
        done < "$1"

5. Criação dos webhooks para CI com jenkins:

        #!/bin/bash
        # run as: ./script5.sh lista.txt

        prefix="disciplinas_"
        organization="uspdev-labs"
        jenkins='https://jenkins.uspdev.fflch.usp.br/github-webhook/'

        while IFS='' read -r line || [[ -n "$line" ]]; do
          curl -H "Authorization: token $token" https://api.github.com/repos/$organization/$prefix$line/hooks -d '{"name": "web", "config": { "url": "'$jenkins'", "content_type": "json"}}'
        done < "$1"

6. Adicionar uma task no playbook para criação de todos bancos de dados de lista.txt

7. Adicionar uma task no playbook para criação dos jobs no jenkins




