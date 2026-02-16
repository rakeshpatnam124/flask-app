environment {
  HOST   = "13.201.40.132"
  USER   = "ec2-user"
  APPDIR = "/var/www/flaskapp"
  KEY    = "/var/lib/jenkins/.ssh/ec2-key.pem"
}

stage('Deploy') {
  steps {
    sh '''
      tar --warning=no-file-changed --exclude=.git --exclude=venv --exclude=__pycache__ \
          -czf /tmp/app.tgz .

      scp -i $KEY -o StrictHostKeyChecking=no /tmp/app.tgz $USER@$HOST:/tmp/app.tgz

      ssh -i $KEY -o StrictHostKeyChecking=no $USER@$HOST "
        set -e

        # SAFETY CHECK (prevents deleting / or /usr etc.)
        if [ '$APPDIR' != '/var/www/flaskapp' ]; then
          echo 'APPDIR is unsafe: $APPDIR'
          exit 1
        fi

        mkdir -p '$APPDIR'
        rm -rf '$APPDIR'/*
        tar -xzf /tmp/app.tgz -C '$APPDIR'

        cd '$APPDIR'
        python3 -m venv venv
        source venv/bin/activate
        pip install -U pip
        pip install -r requirements.txt

        pkill -f gunicorn || true
        nohup venv/bin/gunicorn -b 0.0.0.0:8000 app:app >/tmp/gunicorn.log 2>&1 &
      "
    '''
  }
}
