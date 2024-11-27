- First get a pass for an smtp app.
- https://myaccount.google.com/apppasswords


```
sendemail -f from@gmail.com \
          -t tomail@gmail.com \
          -u "Test Email" \
          -m "Bu bir test e-postasıdır." \
          -s smtp.gmail.com:587 \
          -xu mail@gmail.com \
          -xp apppassfromlink \
          -o tls=yes
```
