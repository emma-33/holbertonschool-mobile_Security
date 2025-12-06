adb -s emulator-5554 install app-release-task2.apk 
adb shell settings put global http_proxy 10.5.0.22:8080

pioupiou@oem-Latitude-7480:~/cyber-spe$ adb shell settings put global http_proxy 10.5.0.22:8080
pioupiou@oem-Latitude-7480:~/cyber-spe$ openssl x509 -inform DER -in certificate.der -out burp_cert.pem

pioupiou@oem-Latitude-7480:~/cyber-spe$ openssl x509 -inform PEM -subject_hash_old -in burp_cert.pem | head -1
9a5ba575
pioupiou@oem-Latitude-7480:~/cyber-spe$ mv burp_cert.pem 9a5ba575.0
pioupiou@oem-Latitude-7480:~/cyber-spe$ adb push 9a5ba575.0 /sdcard
9a5ba575.0: 1 file pushed. 0.1 MB/s (1326 bytes in 0.025s)
