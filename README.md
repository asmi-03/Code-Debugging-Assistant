# Code-Debugging-Assistant
import subprocess
import openai
import os
from dotenv import load_dotenv

# Load environment variables from .env
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

def run_code(file_path):
    try:
        result = subprocess.run(
            ["python", file_path],
            capture_output=True,
            text=True,
            timeout=10
        )
        return result.stdout, result.stderr
    except subprocess.TimeoutExpired:
        return "", "Error: Script timed out after 10 seconds."
    except Exception as e:
        return "", f"Unexpected error: {str(e)}"

def ask_gpt_to_debug(code, error):
    prompt = f"""
You are a Python debugging assistant.
The following code has an error. Please explain the error and fix the code.

Code:

Error:

Return only the fixed code and a short explanation.
"""
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are a helpful Python code fixer."},
            {"role": "user", "content": prompt}
        ]
    )
    return response.choices[0].message.content.strip()

def main():
    file_path = input("Enter the path to your Python script: ").strip()

    if not os.path.isfile(file_path):
        print(f"❌ File not found: {file_path}")
        return

    print("▶ Running code...\n")
    stdout, stderr = run_code(file_path)

    if stderr:
        print("❌ Error detected:\n")
        print(stderr)
        print("\n🤖 Asking GPT for suggestions...\n")
        with open(file_path, 'r') as f:
            code = f.read()
        suggestion = ask_gpt_to_debug(code, stderr)
        print("✅ GPT Suggestion:\n")
        print(suggestion)
    else:
        print("✅ Code ran successfully:\n")
        print(stdout)

if __name__ == "__main__":
    main()

