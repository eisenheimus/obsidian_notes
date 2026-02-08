echo "# test" >> README.md 
git init 
git add README.md 
git commit -m "first commit" 
git branch -M main 
git remote add origin https://github.com/eisenheimus/test.git git push -u origin main 


git remote add origin https://github.com/eisenheimus/test.git 
git branch -M main 
git push -u origin main

# Добавьте ключ в агент
ssh-add ~/.ssh/git

# Проверьте подключение
ssh -T git@github.com

![[Pasted image 20260208003458.png]]