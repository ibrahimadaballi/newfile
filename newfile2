import requests, sys, json, uuid, time, os, threading, re, shutil
from termcolor import colored
from colorama import init, Fore, Back, Style  # colorama import

# === Terminal temizleme ve renkleri başlat ===
os.system('cls' if os.name=='nt' else 'clear')
init(autoreset=True)

# === Terminal renkleri ===
Y = '\033[92m'
K = '\033[91m'
M = '\033[95m'
U = '\033[93m'

# === cfonts banner ===
try:
    from cfonts import render
except ImportError:
    os.system('pip install python-cfonts')
    from cfonts import render

# === Terminal boyutu ===
term_width, _ = shutil.get_terminal_size()

# === Banner ===
banner = render('baron', colors=('white','red'), align='center')
print(banner)

# === Banner altına TikTok Otomasyon Botu Loading... ===
bot_text = "TikTok Otomasyon Botu " + colored("Loading...", "green", attrs=["bold"])
print(bot_text.center(term_width))  # Ortala

# === Alt çubuk ===
print(colored("=" * term_width, "cyan"))

API = "https://zefame-free.com/api_free.php?action=config"

names = {
    229: "TikTok Görüntüleme",
    228: "TikTok Takipçi",
    232: "TikTok Beğeni",
    235: "TikTok Paylaş",
    236: "TikTok Kaydetme"
}

# === Servisleri al ===
data = requests.get(API).json()
services = data.get('data', {}).get('tiktok', {}).get('services', [])

# === Servisleri listele ===
for i, service in enumerate(services, 1):
    sid = service.get('id')
    name = names.get(sid, service.get('name', '').strip())
    desc = service.get('description', '').strip()
    status = colored("[ÇALIŞIYOR]", "green") if service.get('available') else colored("[DURDURULDU]", "red")
    print(colored(f"{i}. ", "yellow") + colored(f"{name:<20}", "cyan") + f" {status} " + colored(f"[{desc}]", "magenta"))

# === Seçim ===
choice = input(colored("\nBir veya birden fazla sayı gir (virgül ile ayır): ", "yellow")).strip()
selected_indices = [int(x) for x in choice.split(",") if x.strip().isdigit()]
selected_services = [services[i - 1] for i in selected_indices if 1 <= i <= len(services)]

# === Link al ===
has_follow = any(s.get('id') == 228 for s in selected_services)
if has_follow:
    link = input(colored("Kullanıcı Linki (https://www.tiktok.com/@username): ", "yellow")).strip()
else:
    link = input(colored("Video Linki: ", "yellow")).strip()

# === Video ID ===
video_id = None
if not has_follow:
    try:
        resp = requests.post("https://zefame-free.com/api_free.php?", data={"action": "checkVideoId", "link": link})
        video_id = resp.json().get("data", {}).get("videoId")
        print(colored(f"Video ID: {video_id}", "green"))
    except:
        print(colored("Video ID alınamadı!", "red"))

# === Geri sayım başlangıç satırı ===
start_line_offset = len(banner.split("\n")) + 3 + len(services) + 2 + 1  # +1: geri sayım için bir satır aşağı

# === Yazı kilidi ===
print_lock = threading.Lock()

def move_cursor(line):
    sys.stdout.write(f"\033[{line};0H")

def clear_line():
    sys.stdout.write(" " * (term_width - 1) + "\r")

def write_line(line, text):
    with print_lock:
        move_cursor(line)
        clear_line()
        sys.stdout.write(text)
        sys.stdout.flush()

# === Görev ===
def run_task(service, line_num):
    sid = service.get('id')
    sname = names.get(sid, service.get('name', 'Bilinmeyen'))
    while True:
        try:
            r = requests.post(
                "https://zefame-free.com/api_free.php?action=order",
                data={"service": sid, "link": link, "uuid": str(uuid.uuid4()), "videoId": video_id or ""}
            ).json()

            msg = r.get("message", "")
            wait = r.get("data", {}).get("nextAvailable")
            wait_seconds = 60

            if wait:
                try:
                    wait = float(wait)
                    if wait > time.time():
                        wait_seconds = int(wait - time.time())
                except:
                    pass
            else:
                match = re.search(r"(\d+)\s*minute[s]?\s*et\s*(\d+)\s*seconde", msg)
                if match:
                    mins, secs = int(match.group(1)), int(match.group(2))
                    wait_seconds = mins * 60 + secs

            wait_seconds = max(wait_seconds + 10, 30)

            for remaining in range(wait_seconds, 0, -1):
                mins, secs = divmod(remaining, 60)
                timer = colored(f"{mins:02d}:{secs:02d}", "yellow", attrs=["bold"])
                write_line(line_num, colored(f"[{sname}] ", "cyan") + timer)
                time.sleep(1)

            write_line(line_num, colored(f"[{sname}] Yeni sipariş gönderiliyor...", "green"))
        except Exception as e:
            write_line(line_num, colored(f"[{sname}] Hata: {e}", "red"))
            time.sleep(30)

# === Thread başlat ===
for idx, s in enumerate(selected_services):
    threading.Thread(target=run_task, args=(s, start_line_offset + idx), daemon=True).start()

# === Sonsuz döngü ===
while True:
    time.sleep(1)
