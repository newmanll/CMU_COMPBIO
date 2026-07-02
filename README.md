# Alignment Bench

A full-stack pairwise sequence alignment app. Paste two DNA/RNA or protein
sequences, pick **Needleman-Wunsch** (global) or **Smith-Waterman** (local),
and see the aligned sequences rendered as color-coded tracks with score and
percent identity.

- **Backend:** Python + Flask, implementing both algorithms from scratch with
  a linear gap penalty. DNA/RNA uses match/mismatch scoring; protein uses
  BLOSUM62.
- **Frontend:** React + Vite, no UI framework — hand-styled.

## Project structure

```
seq-align-app/
├── backend/
│   ├── app.py            # Flask API (POST /api/align)
│   ├── algorithms.py     # Needleman-Wunsch + Smith-Waterman
│   ├── matrices.py       # BLOSUM62 + DNA scoring
│   ├── requirements.txt
│   └── tests/test_algorithms.py
└── frontend/
    ├── src/
    │   ├── App.jsx
    │   ├── api.js
    │   └── components/
    └── package.json
```

## Run locally

**Backend** (Python 3.9+):

```bash
cd backend
python3 -m venv venv && source venv/bin/activate   # optional but recommended
pip install -r requirements.txt
python app.py
# API now running at http://localhost:5000
```

**Frontend** (Node 18+):

```bash
cd frontend
npm install
cp .env.example .env    # defaults to http://localhost:5000, edit if needed
npm run dev
# App now running at http://localhost:5173
```

Open http://localhost:5173, paste two sequences (or click "Load an example
pair"), choose a sequence type and algorithm, and click **Run alignment**.

## Running tests

```bash
cd backend
pip install pytest
pytest -v
```

A GitHub Actions workflow (`.github/workflows/backend-tests.yml`) runs these
automatically on every push that touches `backend/`.

## API

`POST /api/align`

```json
{
  "seq1": "GATTACA",
  "seq2": "GCATGCU",
  "seqType": "dna",       // "dna" | "protein"
  "algorithm": "global",  // "global" | "local"
  "match": 1,
  "mismatch": -1,
  "gap": -2
}
```

Returns `aligned1`, `aligned2`, `match_line`, `score`, `identity_pct`, and
(for local alignments) the matched `range1` / `range2` in the original
sequences. FASTA headers (lines starting with `>`) and whitespace are
stripped automatically.

## Publishing this to GitHub

From inside this folder:

```bash
git init
git add .
git commit -m "Initial commit: Alignment Bench"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

(Create the empty repo on GitHub first, or use `gh repo create` if you have
the GitHub CLI installed.)

## Deploying

- **Frontend:** `npm run build` in `frontend/` produces a static `dist/`
  folder — deploy it to GitHub Pages, Vercel, or Netlify. Set `VITE_API_URL`
  to your deployed backend URL at build time.
- **Backend:** deploy `backend/` to Render, Railway, Fly.io, or any host that
  runs a WSGI app. A `gunicorn` entry point is included in
  `requirements.txt` — e.g. `gunicorn app:app`.

## Notes on the algorithms

- **Needleman-Wunsch (global):** aligns the sequences end to end, using gaps
  wherever needed. Best when the sequences are expected to be similar in
  length and roughly the same overall.
- **Smith-Waterman (local):** finds the highest-scoring matching
  *subsequence*, ignoring dissimilar flanking regions. Best when you expect a
  shared motif or domain inside otherwise unrelated sequences.

Both use a simple linear gap penalty (`gap` per inserted/deleted position)
rather than affine gaps, to keep the implementation easy to read and extend.

## License

MIT — see [LICENSE](LICENSE).
