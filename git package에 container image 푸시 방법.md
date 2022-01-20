1. set up environment variable for git push    
  export CR_PAT=git_token    
  
2. login docker using CR_PAT    
  echo $CR_PAT | docker login ghcr.io -u SML0127 --password-stdin   
  
3. taging exist docker image    
  sudo docker tag container-id ghcr.io/sml0127/pse-worker:tag    
  
4. push docker container(image) to git    
  sudo docker push ghcr.io/sml0127/pse-worker:tag    
