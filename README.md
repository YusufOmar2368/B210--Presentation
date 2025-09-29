CSV_PATH = "/Users/yomar/Downloads/combine.csv"

def find_indices(header_line):
    keys = [h.strip().strip('"').strip("'") for h in header_line.split(',')]
    lower = [k.lower() for k in keys]
    candidates = {
        'height': ['height','ht','playerheight'],
        'weight': ['weight','wt','playerweight'],
        'draft_age': ['ageatdraft','draftage','age_at_draft','age','draft_age','ageatdraftyears']
    }
    idx = {}
    for name, variants in candidates.items():
        for i, k in enumerate(lower):
            for v in variants:
                if v in k:
                    idx[name] = i
                    break
            if name in idx:
                break
    return idx, keys

def safe_float(s):
    if s is None:
        return None
    s = s.strip().strip('"').strip("'")
    if s == "":
        return None
    cleaned = ""
    seen_dot = False
    seen_digit = False
    for ch in s:
        if ch.isdigit():
            cleaned += ch
            seen_digit = True
        elif ch == '.' and not seen_dot:
            cleaned += ch
            seen_dot = True
        elif ch == '-' and not cleaned:
            cleaned += ch
        # ignore other characters (units, commas, spaces)
    if not seen_digit:
        return None
    try:
        return float(cleaned)
    except:
        return None

def compute_averages():
    try:
        f = open(CSV_PATH, 'r', encoding='utf-8')
    except Exception as e:
        print("Error opening file:", e)
        return
    lines = f.readlines()
    f.close()
    if not lines:
        print("File is empty")
        return
    header = lines[0]
    idx_map, keys = find_indices(header)
    sums = {'height': 0.0, 'weight': 0.0, 'draft_age': 0.0}
    counts = {'height': 0, 'weight': 0, 'draft_age': 0}
    for line in lines[1:]:
        if not line.strip():
            continue
        parts = [p.strip() for p in line.rstrip('\n').split(',')]
        for name in ('height', 'weight', 'draft_age'):
            if name in idx_map:
                i = idx_map[name]
                if i < len(parts):
                    v = safe_float(parts[i])
                    if v is not None:
                        sums[name] += v
                        counts[name] += 1
    def avg(name):
        return (sums[name] / counts[name]) if counts[name] > 0 else None
    h = avg('height'); w = avg('weight'); a = avg('draft_age')
    print("Averages (missing/non-numeric values ignored):")
    if h is not None:
        print("  height:", round(h, 3), "(based on %d values)" % counts['height'])
    else:
        print("  height: (column not found or no valid values)")
    if w is not None:
        print("  weight:", round(w, 3), "(based on %d values)" % counts['weight'])
    else:
        print("  weight: (column not found or no valid values)")
    if a is not None:
        print("  draft age:", round(a, 3), "(based on %d values)" % counts['draft_age'])
    else:
        print("  draft age: (column not found or no valid values)")

if __name__ == "__main__":
    compute_averages()# B210--Presentation
