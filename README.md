
Python 3 tool that analyzes photo metadata and generates provenance import os
import json
import subprocess
from datetime import datetime

def exif_read(path):
    """Return EXIF metadata as dict using exiftool JSON output."""
    result = subprocess.run(
        ["exiftool", "-j", "-n", path],
        capture_output=True, text=True
    )
    data = json.loads(result.stdout or "[]")
    return data[0] if data else {}

def parse_dt(s):
    try:
        return datetime.strptime(s, "%Y:%m:%d %H:%M:%S")
    except Exception:
        return None

def provenance_score(meta):
    """Compute confidence score based on metadata integrity."""
    dto = parse_dt(meta.get("DateTimeOriginal", ""))
    cdt = parse_dt(meta.get("CreateDate", ""))
    fmd = parse_dt(meta.get("FileModifyDate", ""))

    score = 0
    reasons = []

    if dto: score += 40; reasons.append("DateTimeOriginal present")
    if cdt: score += 30; reasons.append("CreateDate present")
    if fmd: score += 10; reasons.append("FileModifyDate present")

    if dto and cdt and abs((dto - cdt).total_seconds()) < 120:
        score += 10; reasons.append("CreateDate aligns with DateTimeOriginal")

    if dto and fmd and (fmd < dto):
        score -= 10; reasons.append("FileModifyDate earlier than capture")

    return max(0, min(100, score)), reasons

def anomaly_report(meta):
    """Return list of anomalies with short descriptions."""
    issues = []
    dto = parse_dt(meta.get("DateTimeOriginal", ""))
    cdt = parse_dt(meta.get("CreateDate", ""))
    if dto and cdt and abs((dto - cdt).total_seconds()) > 300:
        issues.append("Timestamp drift: CreateDate differs >5 min")
    if not meta.get("GPSLatitude") or not meta.get("GPSLongitude"):
        issues.append("Missing GPS data")
    if meta.get("Software"):
        issues.append(f"Edited by {meta['Software']}")
    return issues

def build_story(path):
    """Create a human-readable provenance story for a photo."""
    meta = exif_read(path)
    score, reasons = provenance_score(meta)
    anomalies = anomaly_report(meta)

    make = meta.get("Make", "Unknown maker")
    model = meta.get("Model", "Unknown model")
    dto = meta.get("DateTimeOriginal", "Unknown")
    loc = f"{meta.get('GPSLatitude', 'N/A')}, {meta.get('GPSLongitude', 'N/A')}"
    sw = meta.get("Software", "None recorded")

    story = f"""
File: {os.path.basename(path)}
Captured: {dto}
Device: {make} / {model}
Location: {loc}
Integrity score: {score}/100
Signals: {", ".join(reasons) if reasons else "None"}
Anomalies: {", ".join(anomalies) if anomalies else "None detected"}
Software: {sw}
"""
    return story

def save_report(story, filename="provenance_report.txt"):
    """Save the provenance story to a text file."""
    with open(filename, "w", encoding="utf-8") as f:
        f.write(story)
    print(f"\nReport saved to {filename}")

if __name__ == "__main__":
    test_file = input("Enter path to photo: ")
    if os.path.exists(test_file):
        story = build_story(test_file)
        border = "=" * 60
        print(f"\n{border}\n📸 PHOTO PROVENANCE STORY\n{border}")
        print(story)
        print(border)
        save_choice = input("Save report to TXT? (y/n): ").strip().lower()
        if save_choice == "y":
            save_report(story)
    else:
        print("❌ File not found. Please enter a full path to a photo file.")
import os
import json
import subprocess
from datetime import datetime

def exif_read(path):
    """Return EXIF metadata as dict using exiftool JSON output."""
    result = subprocess.run(
        ["exiftool", "-j", "-n", path],
        capture_output=True, text=True
    )
    data = json.loads(result.stdout or "[]")
    return data[0] if data else {}

def parse_dt(s):
    try:
        return datetime.strptime(s, "%Y:%m:%d %H:%M:%S")
    except Exception:
        return None

def provenance_score(meta):
    """Compute confidence score based on metadata integrity."""
    dto = parse_dt(meta.get("DateTimeOriginal", ""))
    cdt = parse_dt(meta.get("CreateDate", ""))
    fmd = parse_dt(meta.get("FileModifyDate", ""))

    score = 0
    reasons = []

    if dto: score += 40; reasons.append("DateTimeOriginal present")
    if cdt: score += 30; reasons.append("CreateDate present")
    if fmd: score += 10; reasons.append("FileModifyDate present")

    if dto and cdt and abs((dto - cdt).total_seconds()) < 120:
        score += 10; reasons.append("CreateDate aligns with DateTimeOriginal")

    if dto and fmd and (fmd < dto):
        score -= 10; reasons.append("FileModifyDate earlier than capture")

    return max(0, min(100, score)), reasons

def anomaly_report(meta):
    """Return list of anomalies with short descriptions."""
    issues = []
    dto = parse_dt(meta.get("DateTimeOriginal", ""))
    cdt = parse_dt(meta.get("CreateDate", ""))
    if dto and cdt and abs((dto - cdt).total_seconds()) > 300:
        issues.append("Timestamp drift: CreateDate differs >5 min")
    if not meta.get("GPSLatitude") or not meta.get("GPSLongitude"):
        issues.append("Missing GPS data")
    if meta.get("Software"):
        issues.append(f"Edited by {meta['Software']}")
    return issues

def build_story(path):
    """Create a human-readable provenance story for a photo."""
    meta = exif_read(path)
    score, reasons = provenance_score(meta)
    anomalies = anomaly_report(meta)

    make = meta.get("Make", "Unknown maker")
    model = meta.get("Model", "Unknown model")
    dto = meta.get("DateTimeOriginal", "Unknown")
    loc = f"{meta.get('GPSLatitude', 'N/A')}, {meta.get('GPSLongitude', 'N/A')}"
    sw = meta.get("Software", "None recorded")

    story = f"""
File: {os.path.basename(path)}
Captured: {dto}
Device: {make} / {model}
Location: {loc}
Integrity score: {score}/100
Signals: {", ".join(reasons) if reasons else "None"}
Anomalies: {", ".join(anomalies) if anomalies else "None detected"}
Software: {sw}
"""
    return story

def save_report(story, filename="provenance_report.txt"):
    """Save the provenance story to a text file."""
    with open(filename, "w", encoding="utf-8") as f:
        f.write(story)
    print(f"\nReport saved to {filename}")

if __name__ == "__main__":
    test_file = input("Enter path to photo: ")
    if os.path.exists(test_file):
        story = build_story(test_file)
        border = "=" * 60
        print(f"\n{border}\n📸 PHOTO PROVENANCE STORY\n{border}")
        print(story)
        print(border)
        save_choice = input("Save report to TXT? (y/n): ").strip().lower()
        if save_choice == "y":
            Just another day
            
