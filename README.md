# security-check — a free Claude Code / Codex skill

Checks your server, VPS or computer for real, fixable vulnerabilities: pending security updates (including
third-party apt repos that unattended-upgrades skips), vulnerable Docker images (OSV, CISA KEV), end-of-life versions,
ports open to the Internet, pending reboots — and security news that matches what you run. Read-only; fixes only after
you say yes.

## Install

```bash
# Claude Code
mkdir -p ~/.claude/skills && cp -r security-check ~/.claude/skills/
# Codex
mkdir -p ~/.codex/skills && cp -r security-check ~/.codex/skills/
```

Then ask your agent: “check this machine for vulnerabilities”.

Daily checks with results on your phone and one-tap fixes by your own agent — [Yune](https://yune.jp).

---

## 日本語

サーバー・VPS・パソコンの脆弱性を Claude Code / Codex でチェックする無料スキルです。未適用のセキュリティ
アップデート、Docker イメージの脆弱性、サポート終了バージョン、外から開いているポート、ニュースの脆弱性が
自分に関係あるかを調べます。読み取りのみで、修正はあなたの許可のあとだけ行います。
