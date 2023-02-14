# DO280-apps

Inicio em 13/02/23
DO280 - Red Hat OpenShift Administration II: Operating a Production Kubernetes Cluster

[student@workstation ~]$ ssh lab@utility
[lab@utility ~]$ ./wait.sh

## Pergutas e Respostas

	# 1. Qual das definições a seguir melhor descreve as plataformas de orquestração de contêineres?
	
		R: Elas permitem que você gerencie um cluster de servidores que execute aplicativos em contêineres. 
			Elas adicionam recursos, como autosserviço, alta disponibilidade, monitoramento e automação.
			
	# 2. Quais dos recursos principais a seguir permitem alta disponibilidade para seus aplicativos? (Escolha três opções.)
	
		R: Os balanceadores de carga do OpenShift HAProxy permitem acesso externo aos aplicativos.
		   Os serviços do OpenShift equilibram o acesso a aplicativos de dentro do cluster.
		   As configurações de implantação do OpenShift garantem que os contêineres de aplicativos 
			sejam reiniciados em cenários como a perda de um nó.
			
	# 3. Quais das instruções a seguir sobre o OpenShift estão corretas? (Escolha duas opções.)
	
		R: Os desenvolvedores podem criar e iniciar aplicativos de nuvem diretamente de um repositório de código-fonte.
		   Os administradores de clusters do OpenShift podem descobrir e instalar novos operadores no catálogo do operador.
		   
	# 4. Quais dos serviços a seguir os componentes do OpenShift usam para balancear a carga do seu tráfego? (Escolha duas opções.)
	
		R: Serviços, que usam o netfilter para balanceamento de carga.
		   Rotas, que usam o HAProxy para balanceamento de carga.
		   
	# 5. Quais das instruções a seguir sobre a alta disponibilidade e o dimensionamento do OpenShift estão corretas? (Escolha duas opções.)
	
		R: O OpenShift usa métricas do Prometheus para dimensionar dinamicamente os pods do aplicativo.
		   O OpenShift pode ampliar e reduzir verticalmente o dimensionamento de acordo com a demanda.


	# 1. O OpenShift é baseado em quais das tecnologias de orquestração de contêineres a seguir?
	
		R: Kubernetes
		
	# 2. Quais das instruções a seguir sobre o OpenShift Container Platform estão corretas? (Escolha duas opções.)
	
		R: O OpenShift fornece um servidor OAuth que autentica chamadas para sua API REST.
		   O OpenShift exige um engine de contêiner CRI-O.
	
	# 3. Qual dos servidores a seguir executa componentes de API do Kubernetes?
	
		R: Nós de plano de controle
		
	# 4. Qual dos componentes a seguir o OpenShift adiciona ao Kubernetes upstream?
	
		R: Um servidor de registro
		
	# 5. Qual das sentenças a seguir é verdadeira em relação ao suporte para armazenamento com o OpenShift?
	
		R: Os usuários podem implantar aplicativos que exigem armazenamento persistente, contando com a classe de armazenamento padrão.


	# 1. Qual dos seguintes métodos de instalação requer o uso do instalador do OpenShift para configurar nós de plano de controle e de computação?
	
		R: Automação de pilha completa e infraestrutura preexistente.
	
	# 2. Qual dos seguintes métodos de instalação permite a configuração de nós usando o Red Hat Enterprise Linux?
	
		R: Infraestrutura preexistente.
		
	# 3. Qual dos métodos de instalação a seguir permite o uso de um provedor de virtualização não compatível com a perda de alguns recursos do 
		 OpenShift?
		 
		R: Infraestrutura preexistente.

	# 4. Qual método de instalação permite o uso de vários provedores de nuvem compatíveis com o mínimo de esforço?
	
		R: Automação de pilha completa.
		
	# 5. Qual dos métodos de instalação a seguir permite a ampla personalização das configurações do cluster, fornecendo entrada para o instalador 
		 do OpenShift?
	
		R: Nem automação de pilha completa nem infraestrutura preexistente.
		

## Verificando nodes
	
	oc get nodes
	
## Vendo o uso real de CPU e memória dos seus nós
	
	oc adm top node
	
## Use o comando oc describe para verificar se todas as condições que podem indicar problemas são falsas.
	
	oc describe node <NAME>
	
## Liste todos os pods dentro do projeto openshift-image-registry
	
	oc get pod -n openshift-image-registry
	
## Siga os logs do pod do operador (cluster-image-registry-operator-xxx)
	
	oc logs --tail 3 -n openshift-image-registry <POD_NAME>
	
## Siga os logs do Kubelet do mesmo nó que você inspecionou para uso de CPU e memória na etapa anterior.

	oc adm node-logs --tail 1 -u kubelet master01
	

## Inicie uma sessão do shell no nó
	
	oc debug node/<NAME>
	
## Liste todos os eventos do projeto atual 

	oc get events
	
## Use o Skopeo para encontrar informações sobre a imagem de contêiner dos evento

	skopeo inspect docker://registry.redhat.io/rhel8/postgresql-13:1
	
## Login no Kubeadmin (API - URL)
	
	oc login -u kubeadmin -p <PASSWORD> <URL:PORT>
	
## Verifique a classe de armazenamento padrão.

	oc get storageclass
	
## Crie uma nova implantação de banco de dados usando a imagem de contêiner localizada em registry.redhat.io/rhel8/postgresql-13:1-7.

	oc new-app --name postgresql-persistent \
	--image registry.redhat.io/rhel8/postgresql-13:1-7 \
	-e POSTGRESQL_USER=redhat \
	-e POSTGRESQL_PASSWORD=redhat123 \
	-e POSTGRESQL_DATABASE=persistentdb

## Crie uma nova solicitação de volume persistente para adicionar um novo volume à implantação postgresql-persistent.

	oc set volumes deployment/postgresql-persistent \
	--add --name postgresql-storage --type pvc --claim-class nfs-storage \
	--claim-mode rwo --claim-size 10Gi --mount-path /var/lib/pgsql \
	--claim-name postgresql-storage
	
## Use o comando htpasswd para preencher o arquivo de autenticação HTPasswd com os nomes de usuários e senhas criptografadas. 
	A opção -B usa criptografia bcrypt. Por padrão, o comando htpasswd usa o algoritmo de hash MD5 quando você não especifica 
	outro algoritmo.

	htpasswd -c -B -b ~/DO280/labs/auth-provider/htpasswd admin redhat
	
## Atribua o usuário admin à função cluster-admin.

	oc adm policy add-cluster-role-to-user cluster-admin admin
	
## Liste os usuários atuais

	oc get users
	
## Exiba a lista de identidades atuais
	
	oc get identity
	
## Gere uma senha de usuário aleatória e a atribua à variável MANAGER_PASSWD

    MANAGER_PASSWD="$(openssl rand -hex 15)"

## Edite o recurso no local para remover o provedor de identidade de OAauth

    oc edit oauth

## Exclua todos os recursos de usuário

    oc delete user --all

## Exclua todos os recursos de identidade

    oc delete identity --all