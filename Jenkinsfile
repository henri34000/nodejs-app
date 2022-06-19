

pipeline {

        // We will use any agent available.
        agent any

        // set a timeout of 30 minutes for this pipeline. 
        options
        {
            timeout(time: 30, unit: 'MINUTES')
        }

        // List of parameters required to configure before the launch of the pipeline.
        parameters{
            string(name: 'BRANCH', defaultValue: 'master', description: 'Name of the branch to use.')
            choice(name: 'ENVIRONMENT', choices: ['issam-mejri-ext-dev', 'issam-mejri-ext-stage'], description: 'The name of the environement where we want to deploy/build resources.')
            choice(name: 'DEPLOYMENT_TYPE', choices: ['','BUILD', 'DEPLOY'], description: 'The name of the type of deployment to process, can be either Build or Deploy.')
            string(name: 'SOURCES_URL', defaultValue: 'https://github.com/imejri/nodejs-app.git', description: 'Gitlab repository source of the application.')

        }

        // List of stages used on the pipelines.
        stages
        {
            stage('Checkout.')
            {
                steps
                {
                    script{
                            if ( params.DEPLOYMENT_TYPE.isEmpty() )
                            {
                                // Use SUCCESS FAILURE or ABORTED
                                currentBuild.result = "FAILURE"
                                throw new Exception("The Parameters DEPLOYMENT_TYPE is empty, the value must be either BUILD or DEPLOY, Please set DEPLOYMENT_TYPE parameters.")
                            }
                    }
                        
                    checkout([$class: 'GitSCM', 
                            branches: [[name: "*/${env.BRANCH}"]], 
                            userRemoteConfigs:[[credentialsId:  'gitlab', url: "${env.SOURCES_URL}"]]
                    ])
                    
                } // steps
            } // stage

            stage("Build Source Code")
            {
                when {
                    // Only for BUILD Deployment type.
                    expression { params.DEPLOYMENT_TYPE == 'BUILD' }
                }
                steps
                {

                    script
                    {
                       sh """
                        oc new-build --name=nodejs-image --image-stream=openshift/nodejs:16-ubi8 ${params.SOURCES_URL} --to='nodejs-image:${GIT_COMMIT}'
                        """
                    } // script 
                } // steps
            } // stage   

            stage("Deploy resources for DEV/STAGE environments.")
            {
                 when {
                    expression { params.DEPLOYMENT_TYPE == 'DEPLOY' }
                 }
                steps
                { 
                    script
                    {
                       sh """
                        oc apply -f nodejs-image-demo-deploy.yaml -n ${params.ENVIRONMENT}
                        oc apply -f nodejs-image-demo-svc.yaml -n ${params.ENVIRONMENT}
                        """
                    } // script
                } // steps
            } // stage

		stage("Create route to access the Application")
            	{
                 when {
                    expression { params.ENVIRONMENT == 'issam-mejri-ext-dev' && params.DEPLOYMENT_TYPE == 'DEPLOY' }
                 }
                steps
                { 
                    script
                    {
			sh """
                       	   oc apply -f nodejs-image-demo-route.yaml -n ${params.ENVIRONMENT}
			"""
                    } // script
                } // steps
            } // stage
        } // stages

        post 
        {
            always 
            {
                deleteDir() // Delete the current workspace.
                dir("${env.WORKSPACE}@script") 
                { 
                    deleteDir() // Delete the script workspace, this allow us to update the pipeline script without deleting the builconfig.
                } // dir
            } // always
        } // post 
} // pipeline
