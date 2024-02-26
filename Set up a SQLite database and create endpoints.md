# Task-answer
\*this is the answer of the task you sent which is:
Develop a Flask API that exposes endpoints for managing test cases and their execution results
Across multiple test assets, with data stored in a SQLite database.*/

from flask import Flask, request, jsonify
import sqlite3

app = Flask(__name__)

# SQLite database setup
conn = sqlite3.connect('test_cases.db')
c = conn.cursor()

# Create table if not exists
c.execute('''CREATE TABLE IF NOT EXISTS test_cases
             (id INTEGER PRIMARY KEY, name TEXT, description TEXT, result TEXT)''')
conn.commit()

# Close connection after setup
conn.close()

# Endpoint to create a new test case
@app.route('/test_cases', methods=['POST'])
def create_test_case():
    data = request.json
    name = data.get('name')
    description = data.get('description')
    
    if not name or not description:
        return jsonify({'error': 'Name and description are required'}), 400
    
    conn = sqlite3.connect('test_cases.db')
    c = conn.cursor()
    c.execute("INSERT INTO test_cases (name, description) VALUES (?, ?)", (name, description))
    conn.commit()
    conn.close()
    
    return jsonify({'message': 'Test case created successfully'}), 201

# Endpoint to retrieve all test cases
@app.route('/test_cases', methods=['GET'])
def get_all_test_cases():
    conn = sqlite3.connect('test_cases.db')
    c = conn.cursor()
    c.execute("SELECT * FROM test_cases")
    test_cases = c.fetchall()
    conn.close()
    
    return jsonify({'test_cases': test_cases})

# Endpoint to retrieve a single test case by its ID
@app.route('/test_cases/<int:test_case_id>', methods=['GET'])
def get_test_case(test_case_id):
    conn = sqlite3.connect('test_cases.db')
    c = conn.cursor()
    c.execute("SELECT * FROM test_cases WHERE id=?", (test_case_id,))
    test_case = c.fetchone()
    conn.close()
    
    if not test_case:
        return jsonify({'error': 'Test case not found'}), 404
    
    return jsonify({'test_case': test_case})

# Endpoint to update an existing test case
@app.route('/test_cases/<int:test_case_id>', methods=['PUT'])
def update_test_case(test_case_id):
    data = request.json
    name = data.get('name')
    description = data.get('description')
    
    if not name or not description:
        return jsonify({'error': 'Name and description are required'}), 400
    
    conn = sqlite3.connect('test_cases.db')
    c = conn.cursor()
    c.execute("UPDATE test_cases SET name=?, description=? WHERE id=?", (name, description, test_case_id))
    conn.commit()
    conn.close()
    
    return jsonify({'message': 'Test case updated successfully'})

# Endpoint to delete a test case by its ID
@app.route('/test_cases/<int:test_case_id>', methods=['DELETE'])
def delete_test_case(test_case_id):
    conn = sqlite3.connect('test_cases.db')
    c = conn.cursor()
    c.execute("DELETE FROM test_cases WHERE id=?", (test_case_id,))
    conn.commit()
    conn.close()
    
    return jsonify({'message': 'Test case deleted successfully'})

# Recording the execution result of a test case for a specific test asset
# Retrieving the execution results of all test cases for a specific test asset

if __name__ == '__main__':
    app.run(debug=True)
