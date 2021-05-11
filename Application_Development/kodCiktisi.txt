import mysql.connector

mydb = mysql.connector.connect(
  host="localhost",
  user="root",
  password="",
  database="demo" #databasemizin ismini bağlanmak için veriyoruz.
)

gold_class_percentage=0.25
silver_class_percentage=0.5


# tüm customer tablosunu okuyup passaportları listeye attım
mycursor = mydb.cursor()
mycursor.execute("SELECT FFC.Passaport_no, SUM(flight_leg.Mileage) FROM ffc,flight_leg WHERE ffc.Flight_number = flight_leg.Flight_Number AND flight_leg.Leg_Number = ffc.Leg_number GROUP BY ffc.Passaport_no;")
myresult = mycursor.fetchall()




mycursor.execute("CREATE TABLE IF NOT EXISTS customerSegmentation(passaport_no int NOT NULL,class varchar(255),FOREIGN KEY (passaport_no) REFERENCES customer(Passaport_no),PRIMARY KEY (passaport_no));")
mycursor.execute("TRUNCATE customerSegmentation")


n = len(myresult)


for i in range(n - 1):

  for j in range(0, n - i - 1):


    if myresult[j][1] > myresult[j + 1][1]:
      myresult[j], myresult[j + 1] = myresult[j + 1], myresult[j]










sql = "INSERT INTO customerSegmentation VALUES ('{0}','{1}');"
maxCustomer = len(myresult)
for i in range(round(n*gold_class_percentage)):
  mycursor.execute("INSERT INTO customerSegmentation VALUES ('{0}','{1}');".format(myresult[maxCustomer-1][0],'GOLD'))
  maxCustomer-=1
  mydb.commit()

for i in range(round((n*silver_class_percentage)-(n*gold_class_percentage))):
  mycursor.execute("INSERT INTO customerSegmentation VALUES ('{0}','{1}');".format(myresult[maxCustomer-1][0],'SILVER'))
  maxCustomer-=1
  mydb.commit()


