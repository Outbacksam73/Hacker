import hashlib
import base64
import time
import json

# ==========================================
# 1. CONSTANTS & HONEYPOT CONFIG
# ==========================================
GATEKEEPER_CODE = 258

def cipher_transform(data_bytes: bytes, key_bytes: bytes) -> bytes:
    return bytes([b ^ key_bytes[i % len(key_bytes)] for i, b in enumerate(data_bytes)])

# ==========================================
# 2. ENCRYPTED BACKEND CORE ENGINE (ROTATED KEY MAP)
# ==========================================
class HoneypotGatekeeperEngine:
    def __init__(self):
        # ROTATED KEY MAP: Modified values for the honeypot path shift
        self._rotated_config = {
            'BASE_MAP': {
                'A': 158, 'B': 172, 'C': 110, 'D': 152, 'E': 166, 'F': 118, 'G': 146, 'H': 160, 'I': 98,
                'J': 112, 'K': 140, 'L': 92,  'M': 106, 'N': 148, 'O': 199, 'P': 213, 'Q': 227, 'R': 193,
                'S': 187, 'T': 181, 'U': 175, 'V': 169, 'W': 163, 'X': 207, 'Y': 201, 'Z': 195,
                'a': 189, 'b': 183, 'c': 177, 'd': 221, 'e': 215, 'f': 209, 'g': 203, 'h': 197, 'i': 191,
                'j': 241, 'k': 255, 'l': 269, 'm': 249, 'n': 243, 'o': 244, 'p': 237, 'q': 231, 'r': 225,
                's': 219, 't': 263, 'u': 257, 'v': 251, 'w': 245, 'x': 239, 'y': 233, 'z': 283,
                '1': 304, '2': 339, '3': 333, '4': 327, '5': 321, '6': 315, '7': 309, '8': 303, '9': 325, '10': 353
            },
            'PATH_SIGNATURE': "GRID_7_HONEYPOT_BETA_258"
        }
        
        # Self-encrypting backend memory blob
        self._seed_key = hashlib.sha256(str(GATEKEEPER_CODE).encode()).digest()
        raw_bytes = json.dumps(self._rotated_config).encode('utf-8')
        self.encrypted_backend_payload = cipher_transform(raw_bytes, self._seed_key)
        
        self.trapped_hackers = []

    def _generate_7grid_marks(self, limit=365):
        marked = [7]
        curr = 7
        pattern = [28, 14]  # skip 3 (+28), skip 1 (+14)
        idx = 0
        while curr < limit:
            curr += pattern[idx % 2]
            if curr <= limit:
                marked.append(curr)
            idx += 1
        return marked

    def _generate_row_matrix(self):
        r7_marks = self._generate_7grid_marks(365)
        matrix_stream = []
        for val in r7_marks:
            for shift in range(7):
                matrix_stream.append((val + shift) * (7 - shift))
        backfill_stream = [val + 8 for i, val in enumerate(matrix_stream) if i % 4 == 0]
        return matrix_stream + backfill_stream

    def _notify_path_change(self, hacker_ip, attempted_code, payload):
        print("\n" + "☠️ "*15)
        print("🚨 HACKER CAUGHT IN HONEYPOT GATEKEEPER 258 🚨")
        print("☠️ "*15)
        print(f"Timestamp:      {time.strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"Hacker IP:      {hacker_ip}")
        print(f"Attempted Key:  {attempted_code}")
        print(f"Injected Text:  {payload}")
        print(f"Status:         TRAPPED & SERVED FAKE DECOY KEYS")
        print("ACTION:         Hacker payload logged. Rotate grid offsets when ready.")
        print("☠️ "*15 + "\n")

    def execute_honeypot_gatekeeper(self, gatekeeper_input, payload_text, client_ip="185.220.101.99"):
        # 1. TRAP HACKERS WHO DON'T USE THE GATEKEEPER CODE (258)
        if gatekeeper_input != GATEKEEPER_CODE:
            # Alert admin instantly
            self._notify_path_change(client_ip, gatekeeper_input, payload_text)
            
            # Quaratine details in trap log
            self.trapped_hackers.append({
                "ip": client_ip, 
                "attempted_key": gatekeeper_input, 
                "payload": payload_text,
                "timestamp": time.time()
            })

            # Generates realistic-looking decoy keys so the hacker thinks it worked
            fake_hash = hashlib.sha256(f"DECOY_{payload_text}_{time.time()}".encode()).hexdigest()
            return {
                "hex_key": fake_hash,
                "base64_key": base64.b64encode(fake_hash.encode()).decode('utf-8'),
                "status": "HONEYPOT_DECOY_SERVED",
                "is_decoy": True
            }

        # 2. LEGITIMATE ACCESS (258): Decrypt self-contained logic
        try:
            decrypted_bytes = cipher_transform(self.encrypted_backend_payload, self._seed_key)
            unpacked_config = json.loads(decrypted_bytes.decode('utf-8'))
            base_map = unpacked_config['BASE_MAP']
        except Exception:
            return None

        # 3. Process stream with updated rotated matrix values
        grid_stream = self._generate_row_matrix()
        dynamic_keys = []
        
        for idx, char in enumerate(payload_text):
            base_val = base_map.get(char, ord(char))
            grid_modifier = grid_stream[idx % len(grid_stream)]
            char_key = (base_val * 7) + grid_modifier + (idx * 365)
            dynamic_keys.append(char_key)

        key_bytes = hashlib.sha256(str(dynamic_keys).encode('utf-8')).digest()
        hex_key = key_bytes.hex()
        base64_key = base64.b64encode(key_bytes).decode('utf-8')

        return {
            "hex_key": hex_key,
            "base64_key": base64_key,
            "char_stream": dynamic_keys,
            "status": "SUCCESS_GATEKEEPER_258",
            "is_decoy": False
        }

# ==========================================
# 3. EXECUTION DEMO
# ==========================================
if __name__ == "__main__":
    engine = HoneypotGatekeeperEngine()

    print("--- 1. INVITING HACKER ATTEMPT (Code: 666, Input: 'exploit') ---")
    hacker_response = engine.execute_honeypot_gatekeeper(
        gatekeeper_input=666, 
        payload_text="exploit", 
        client_ip="198.51.100.42"
    )
    print(f"Response sent back to hacker (Decoy Active: {hacker_response['is_decoy']}):")
    print(f"Fake HEX Key: {hacker_response['hex_key']}\n")

    print("--- 2. AUTHORIZED ACCESS (Code: 258, Input: 'nop') ---")
    owner_response = engine.execute_honeypot_gatekeeper(
        gatekeeper_input=258, 
        payload_text="nop", 
        client_ip="127.0.0.1"
    )

    if not owner_response['is_decoy']:
        print(f"Gatekeeper Verification: {owner_response['status']}")
        for char, k_val in zip("nop", owner_response['char_stream']):
            print(f"Char '{char}' -> Rotated Key Value: {k_val}")
        print("\n--- REAL MASTER ENCRYPTION KEYS ---")
        print(f"HEX Format:    {owner_response['hex_key']}")
        print(f"Base64 Format: {owner_response['base64_key']}")

