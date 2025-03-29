from pymongo import MongoClient
from dotenv import load_dotenv
from pprint import PrettyPrinter
import os

load_dotenv()

printer = PrettyPrinter()

def get_mongodb_connection():
    mongodb_connection_string = os.environ.get("MONGO_DB_CONNECTION_STRING")
    client = MongoClient(mongodb_connection_string)
    return client.sample_mflix.mcp_unstructured_api_db


def kongapay_september_autoreversals():
    printer = PrettyPrinter()

    transaction_data_collection = get_mongodb_connection()

    result = transaction_data_collection.aggregate([
        {
            "$search": {
                "index": "search-text-index",
                "compound": {  # Requires ALL terms to match
                    "must": [
                        { "text": { "query": "kongapay", "path": "text" } },
                        { "text": { "query": "Auto-Reversal", "path": "text" } },
                        { "text": { "query": "september", "path": "text" } }
                    ]
                }
            }
        },
        {
            "$project": {
                "text": 1,
                "_id": 0
            }
        }
    ])
    results = list(result)
    printer.pprint({
        "metadata": {
            "resource": "transactions://opay/airtime/march",
            "description": "Opay airtime purchases during March"
        },
        "data": results,
        "analysis_prompt": """
            Analyze these Opay airtime purchases and provide:
            1. Total amount spent
            2. Total number of purchases
        """
    })


def opay_march_airtime_purchases(transaction_data_collection):

    transaction_data_collection = get_mongodb_connection()

    result = transaction_data_collection.aggregate([
        {
            "$search": {
                "index": "search-text-index",
                "compound": {  # Requires ALL terms to match
                    "must": [
                        { "text": { "query": "opay", "path": "text" } },
                        { "text": { "query": "airtime", "path": "text" } },
                        { "text": { "query": "march", "path": "text" } }
                    ]
                }
            }
        },
        {
            "$project": {
                "text": 1,
                "_id": 0,
                "score": { "$meta": "searchScore" }
            }
        }
    ])
    printer.pprint(list(result))


def pocketapp_electricity_bill_transactions(transaction_data_collection):

    transaction_data_collection = get_mongodb_connection()

    result = transaction_data_collection.aggregate([
        {
            "$search": {
                "index": "search-text-index",
                "compound": {  # Requires ALL terms to match
                    "must": [
                        { "text": { "query": "pocketapp", "path": "text" } },
                        { "text": { "query": "electricity", "path": "text" } }
                    ]
                }
            }
        },
        {
            "$project": {
                "text": 1,
                "_id": 0,
                "score": { "$meta": "searchScore" }
            }
        }
    ])
    printer.pprint(list(result))

kongapay_september_autoreversals()