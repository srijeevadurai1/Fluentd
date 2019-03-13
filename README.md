To build docker image: 

sudo docker build -t fluentd_img .

NOTE: All required plugins are installed in the images. 

1)	fluent-plugin-postgres-bulk
2)	fluent-plugin-cat-sweep

To launch datareceiver instance of fluentd: 

#sudo docker create --name fluentd_datareceiver -p 30555:30555 -p 31292:9292 fluentd_img

#sudo docker cp test_forward.outputFile.conf fluentd_datareceiver:/fluentd/etc/fluent.conf

=> I have used testing configuration which will write output to file instead of postgres. 
#sudo docker start fluentd_datareceiver


To launch dataforward instance of fluentd: 

#sudo docker create --name fluentd_dataforward -p 31291:9292 --mount type=bind,src=/home/osboxes/Fluent_Data_Input,dst=/home/fluent/csv_input fluentd_img

=> highlighted volume mount has to be decided based on target environment. 

#sudo docker cp test_cat_sweep.fwd.conf fluentd_dataforward:/fluentd/etc/fluent.conf 

=> it is testing configuration, it will read files from /home/fluent/csv_input (mounted volume) and move the processed files into /home/fluent/processed directory.

#sudo docker start fluentd_dataforward
