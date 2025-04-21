import socket
import openai
import time

# === CONFIGURATION ===
TARGET_IP = '127.0.0.1'  # Replace with target IP
PORT_RANGE = (1, 1024)   # Common range; increase if needed
OPENAI_API_KEY = "your-openai-api-key"

openai.api_key = OPENAI_API_KEY

# === FUNCTION TO SCAN PORTS ===
def scan_ports(ip, port_range):
    open_ports = []
    print(f"Scanning {ip} from port {port_range[0]} to {port_range[1]}...")
    for port in range(port_range[0], port_range[1] + 1):
        try:
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
                s.settimeout(0.5)
                result = s.connect_ex((ip, port))
                if result == 0:
                    open_ports.append(port)
        except Exception as e:
            print(f"Error scanning port {port}: {e}")
    return open_ports

# === FUNCTION TO QUERY CHATGPT FOR VULNERABILITIES ===
def get_vulnerabilities_from_chatgpt(port_list):
    prompt = (
        f"I scanned a system and found these open ports: {port_list}. "
        f"Can you list the common vulnerabilities or known risks associated with each port number?"
    )

    try:
        response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo",  # or "gpt-4" if you have access
            messages=[
                {"role": "system", "content": "You are a cybersecurity expert."},
                {"role": "user", "content": prompt}
            ],
            temperature=0.2,
            max_tokens=800
        )
        return response['choices'][0]['message']['content']
    except Exception as e:
        return f"Error calling OpenAI API: {e}"

# === MAIN PROGRAM ===
if __name__ == "__main__":
    start_time = time.time()
    ports = scan_ports(TARGET_IP, PORT_RANGE)

    if ports:
        print(f"\n✅ Open ports found: {ports}")
        print("\n🔍 Querying ChatGPT for vulnerabilities...")
        vuln_info = get_vulnerabilities_from_chatgpt(ports)
        print("\n💡 Vulnerabilities:")
        print(vuln_info)
    else:
        print("❌ No open ports found.")

    print(f"\n⏱️ Scan completed in {round(time.time() - start_time, 2)} seconds.")
