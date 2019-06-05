pipeline {
    agent any
    parameters {
        string(name: 'cliente_id', defaultValue: 'elcometa', description: 'Subdominio del negocio')
        string(name: 'pass_db', defaultValue: 'abcxefg', description: 'Parssword del  schema de bd')
		string(name: 'admin_user', defaultValue: 'poncho', description: 'Usuario administrador de la tienda')
        string(name: 'admin_pass', defaultValue: 'q1w2e3r4', description: 'Password del administrador de  la tienda')
        string(name: 'cajero_user', defaultValue: 'cajero1', description: 'Usuario del cajero')
		string(name: 'cajero_pass', defaultValue: 'q1w2e3r4', description: 'Parssword del cajerjo')
        string(name: 'tae_user', defaultValue: 'Demo', description: 'Usuario de recargas')
        string(name: 'tae_pass', defaultValue: 'q1w2e3r4', description: 'Parssword de recargas')
		string(name: 'nombre_negocio', defaultValue: 'El cometa', description: 'Nombre del negocio')
        string(name: 'giro_negocio', defaultValue: 'Plomeria', description: 'Grio del negocio')
        string(name: 'direccion_negocio', defaultValue: 'Laguna de san cristobal 134', description: 'Direccion del negocio')
		string(name: 'pie_ticket', defaultValue: 'Gracias por su compra', description: 'Pie de pagina del ticket')
		
		booleanParam(name: 'cat_tlapaleria', defaultValue: 'False', description: 'Tlapaleria')
		booleanParam(name: 'cat_papeleria', defaultValue: 'False', description: 'Papeleria')
		booleanParam(name: 'cat_regalos', defaultValue: 'False', description: 'Regalos')		
		booleanParam(name: 'cat_plomeria', defaultValue: 'False', description: 'Plomeria')
		booleanParam(name: 'cat_belleza', defaultValue: 'False', description: 'Belleza')
		
    }
    stages {
        stage('Creacion de scripts ') {
            steps {
                sh "mkdir -p /BuildScript/scripts/${params.cliente_id}/main"
				sh "/BuildScript/crear_schema.sh ${params.cliente_id} ${params.pass_db} "
				sh "/BuildScript/crear_docker_compose.sh ${params.cliente_id} ${params.pass_db} "
				sh "echo ${params.admin_user} > /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.admin_pass} >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.cajero_user}  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.cajero_pass}   >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.tae_user}  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.tae_pass}  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo \\\"${params.nombre_negocio}\\\"  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo \\\"${params.giro_negocio}\\\"  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo \\\"${params.direccion_negocio}\\\"  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo \\\"${params.pie_ticket}\\\"  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt "
				sh "echo ${params.cliente_id}  >> /BuildScript/scripts/${params.cliente_id}/argumentos.txt  "
				sh "xargs --arg-file=/BuildScript/scripts/${params.cliente_id}/argumentos.txt /BuildScript/crear_usuarios.sh "
                sh "/BuildScript/crear_categoria.sh ${params.cliente_id} ${params.cat_tlapaleria}  ${params.cat_papeleria}   ${params.cat_regalos}  ${params.cat_plomeria} ${params.cat_belleza} "				
				sh "echo ${params.cat_plomeria} "
				
				
            }
			
			
        }

       stage('Creando base de datos ') {
            steps {
                sh "/BuildScript/ejecutar_schema.sh ${params.cliente_id} "		
            }	
        }
		
		stage('Creando tablas del sistema ') {
            steps {
                sh "docker run --network=main_default -e CLIENTE_ID=${params.cliente_id}  -e USUARIO_DB=adm_${params.cliente_id} -e PASSWORD_DB=${params.pass_db} --rm fcaudillo/tpv-verde:lts sh -c  " + "'./manage.py makemigrations  && ./manage.py migrate'"		
            }	
        }
		
		stage('Creacion de usuarios ') {
            steps {
                sh "docker run --network=main_default -v /home/dockeradm/data/BuildScript:/BuildScript -e CLIENTE_ID=${params.cliente_id}  -e USUARIO_DB=adm_${params.cliente_id} -e PASSWORD_DB=${params.pass_db} --rm fcaudillo/tpv-verde:lts sh -c " + "'./manage.py  shell< /BuildScript/scripts/${params.cliente_id}/crear_usuarios.py'"		
            }	
        }
		
		stage('Creacion de categorias ') {
            steps {
                sh "docker run --network=main_default -v /home/dockeradm/data/BuildScript:/BuildScript -e CLIENTE_ID=${params.cliente_id}  -e USUARIO_DB=adm_${params.cliente_id} -e PASSWORD_DB=${params.pass_db} --rm fcaudillo/tpv-verde:lts sh -c " + "'./manage.py  shell < /BuildScript/scripts/${params.cliente_id}/crear_categorias.py'"		
            }	
        }
		
		stage('Creacion catalogos TAE ') {
            steps {
                sh "docker run --network=main_default -v /home/dockeradm/data/BuildScript:/BuildScript -e CLIENTE_ID=${params.cliente_id}  -e USUARIO_DB=adm_${params.cliente_id} -e PASSWORD_DB=${params.pass_db} --rm fcaudillo/tpv-verde:lts sh -c " + "'./manage.py  shell< /BuildScript/recargas.py'"		
            }	
        }

       stage('Iniciando el servicio ') {
            steps {
                sh "cd /BuildScript/scripts/${params.cliente_id}/main && docker-compose up -d --no-deps --build ${params.cliente_id} "		
            }	
        }		
		
       stage('Add servicios tpv ') {
            steps {
                sh "cd /BuildScript && ./alta_servicio.sh "		
            }	
        }		

    }
}
