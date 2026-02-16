pipeline {
  agent any

  environment {
    HOST = "13.201.40.132"
    USER = "ec2-user"
    APPDIR = "/var/www/flaskapp"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Deploy') {
      steps {
        sh '''
          tar --warning=no-file-changed \
    --exclude=.git \
    --exclude=venv \
    --exclude=__pycache__ \
    --exclude=app.tgz \
    -czf /tmp/app.tgz .

        scp -o StrictHostKeyChecking=no /tmp/app.tgz $USER@$HOST:/tmp/app.tgz


          ssh -o StrictHostKeyChecking=no $USER@$HOST '
            set -e
            rm -rf $APPDIR/*
            tar -xzf /tmp/app.tgz -C $APPDIR
            cd $APPDIR

            python3 -m venv venv
            source venv/bin/activate
            pip install -U pip
            if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

            pkill -f gunicorn || true
            nohup venv/bin/gunicorn -b 0.0.0.0:8000 app:app >/tmp/gunicorn.log 2>&1 &
          '
        '''
      }
    }
  }
}
