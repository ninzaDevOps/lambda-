mkdir -p layer/python

pip install pymysql cryptography -t layer/python

cd layer
zip -r ../pymysql_layer.zip .
cd ..




code for lambda msql connect
import pymysql
import json
import os
# =========================
# RDS MASTER / WRITER CONFIG
# =========================
RDS_HOST = 'database-1.ccnu6umkc8o4.us-east-1.rds.amazonaws.com'
RDS_USER = 'admin'
RDS_PASSWORD = 'ninza123'
RDS_DB_NAME = 'dev'
TABLE_NAME = 'users'
# =========================
# LAMBDA HANDLER
# =========================
def lambda_handler(event, context):
    connection = None
    try:
        # Connect to RDS
        connection = pymysql.connect(
            host=RDS_HOST,
            user=RDS_USER,
            password=RDS_PASSWORD,
            database=RDS_DB_NAME,
            cursorclass=pymysql.cursors.DictCursor,
            connect_timeout=10
        )
        # Fetch users
        with connection.cursor() as cursor:
            query = f"SELECT * FROM {TABLE_NAME} LIMIT 10"
            cursor.execute(query)
            users = cursor.fetchall()
        # Return response
        return {
            "statusCode": 200,
            "headers": {
                "Content-Type": "application/json"
            },
            "body": json.dumps(users, default=str)
        }
    except Exception as e:
        print("Database error:", str(e))
        return {
            "statusCode": 500,
            "body": json.dumps({
                "error": "Unable to fetch users"
            })
        }
    finally:
        if connection:
            connection.close()
