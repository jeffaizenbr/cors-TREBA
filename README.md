# cors-TREBA


Problemas de CORS
Como a aplicação é disponibilizada em dois módulos, normalmente a implantação ocorre a partir de dois processos distintos que podem, ou não, serem acessíveis por diferentes domínios DNS. Com isso é necessário observar possíveis problemas de CORS (Cross-Origin Resource Sharing).

1. Cookie de sessão
Para manter a sessão de usuário o coyote-backend utiliza um cookie (JSESSIONID), e por isso podem ocorrer problemas quando o coyote-frontend está em um domínio diferente do coyote-backend. Seguem algumas sugestões caso encontre problemas.

1.1. Configurar domínio de validade do cookie
O cookie de sessão é relevante apenas para o coyote-backend, não sendo lido pelo coyote-frontend, então normalmente basta definir o domínio de validade do cookie igual ao domínio onde está o coyote-backend. Com isso o cookie será restrito ao domínio do coyote-backend, sendo enviado pelo navegador em toda a conversação Ajax entre frontend e backend. Esta alteração é feita pela configuração app.sessao.cookie.domain.
Por exemplo, assumindo que:

coyote-frontend: https://coyote.tre-to.jus.br

coyote-backend: https://coyote-api.tre-to.jus.br


basta definir:

app.sessao.cookie.domain=coyote-api.tre-to.jus.br


Caso o domínio do backend seja compartilhado, é possível definir o caminho dentro daquele domínio em que o cookie será válido. Assumindo que:

coyote-frontend: https://coyote.tre-to.jus.br

coyote-backend: https://apps.tre-to.jus.br/coyote


basta definir:

app.sessao.cookie.domain=apps.tre-to.jus.br
app.sessao.cookie.path=/coyote


Com isso será evitado choque com outras aplicações dentro do mesmo domínio.

1.1.1. Detalhes
Nos testes realizados, esta abordagem parece adequada apenas se ambos módulos estiverem em subdomínios do mesmo domínio superior, como nos exemplos acima, em que estão agrupados em tre-to.jus.br. Caso estejam em domínios distintos, alguns navegadores podem exigir a definição da política de SameSite. Atualmente esta configuração não é possível, pois trata de uma política mais atual, não estando prevista na API Servlet usada pelo projeto.
Exemplo de combinação problemática:

coyote-frontend: https://coyote.com.br

coyote-backend: https://coyote-backend.tre-to.jus.br


Outro exemplo:

coyote-frontend: http://localhost

coyote-backend: https://coyote-backend.tre-to.jus.br


Outro detalhe é que, se o coyote-frontend estiver em HTTPS o coyote-backend também precisa estar. Caso contrário o navegador normalmente rejeita o cookie.
Obs.: Testes realizados com Firefox 86.0.1 (aceita domínios distintos) e Chrome 89.0.4389.114 (exige definição de SameSite como None).

1.2. Domínio de validade mais abrangente
Caso a abordagem anterior não funcione, é possível também definir a validade do cookie em todo o domínio superior e então alterar o nome do cookie de sessão, para evitar choque com outras aplicações.
Por exemplo, assumindo que:

coyote-frontend: https://coyote.tre-to.jus.br

coyote-backend: https://coyote-api.tre-to.jus.br


basta definir:

app.sessao.cookie.domain=tre-to.jus.br
app.sessao.cookie.name=coyote-sessionid


Note que neste caso o cookie terá validade em todo o domínio tre-to.jus.br e pode não ser o mais adequado. Avalie primeiro a abordagem anterior, que tende a ser mais adequada e nunca use um domínio mais abrangente que o da sua instituição.

2. Configurações de CORS
Por padrão o coyote-backend permite acesso de todas as origens, não gerando quaisquer problemas. Contudo, caso deseja restringir as regras de CORS, utilize as configurações iniciadas em app.cors conforme descritas na página de parametrização.

3. Proxy Reverso
É possível também evitar qualquer configuração de CORS ou cookie mantendo ambos os módulos no mesmo domínio, normalmente configurando um proxy reverso no servidor web que atende ao coyote-frontend. Veja alguns exemplos abaixo.

3.1. Backend implantado com contexto
Assumindo que:

front-end em: https://coyote.tre-to.jus.br

back-end em: https://coyote-api.tre-to.jus.br/coyote-api


configura-se um proxy reverso da seguinte forma:

/coyote-api => https://coyote-api.tre-to.jus.br/coyote-api


e constrói-se o coyote-frontend indicando como prefixo da API o caminho do proxy reverso:

// URL relativa
base_api_url: '/coyote-api/api/'
// ou absoluta
base_api_url: 'https://coyote-api.tre-to.jus.br/coyote-api/api/'


Obs.: Verifique na documentação do coyote-frontend como configurar este prefixo.

3.2. Backend implantado na raiz do domínio
Assumindo que:

front-end em: https://coyote.tre-to.jus.br

back-end em: https://coyote-api.tre-to.jus.br


configura-se um proxy reverso da seguinte forma:

/api => https://coyote-api.tre-to.jus.br/api


e constrói-se o front-end indicando como prefixo da API o caminho do proxy reverso:

// URL relativa
base_api_url: '/api/'
// ou absoluta
base_api_url: 'https://coyote-api.tre-to.jus.br/api/'
