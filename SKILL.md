---
name: volcengine-video-studio
description: Executable video generation workflow for Volcengine/ARK-compatible video APIs. Use when users need text-to-video, image-to-video, draft-video-to-final-video, ratio/duration/seed controls, task polling, automatic result downloads, or direct task inspection for Volcengine Seedance-style video generation.
---

# volcengine-video-studio

Use this skill to actually submit and complete Volcengine / ARK video generation jobs instead of only drafting prompts.

## Default path

Run the bundled script:

```bash
python3 scripts/generate_video.py "清晨海边，电影感镜头，海风吹动人物衣角，真实光影"
```

By default the script:

- submits the task
- polls until the task finishes
- extracts returned video URLs from the task payload
- downloads generated files into `~/Desktop/volcengine-videos/<timestamp>-<slug>/`

## Required config

The script reads config from env vars:

- `VOLCENGINE_API_KEY` or `ARK_API_KEY`
- `VOLCENGINE_VIDEO_MODEL` (recommended)
- `VOLCENGINE_VIDEO_ENDPOINT` or `VOLCENGINE_ENDPOINT` or `ARK_BASE_URL`

Recommended video models:

- `doubao-seedance-1-0-pro-fast-251015` — default, faster iteration
- `doubao-seedance-1-5-pro-251215` — alternate higher-tier option

Default behavior:

- if `VOLCENGINE_VIDEO_MODEL` is unset, the script defaults to `doubao-seedance-1-0-pro-fast-251015`
- avoid relying on `VOLCENGINE_MODEL` for video runs when that env var is also used for image generation

Typical endpoint:

- `https://ark.cn-beijing.volces.com/api/v3`

The script calls:

- `POST /contents/generations/tasks`
- `GET /contents/generations/tasks/{task_id}`

## Common workflows

### 1. Text to video

```bash
python3 scripts/generate_video.py "赛博朋克城市夜景，镜头缓慢推进，霓虹反射在雨夜街道上" \
  --ratio 16:9 \
  --duration 5
```

### 2. Image to video

```bash
python3 scripts/generate_video.py "让人物微笑并轻微转头，镜头稳定" \
  --image ~/Desktop/portrait.png \
  --ratio 9:16 \
  --duration 5
```

### 3. Draft/sample video to final video

```bash
python3 scripts/generate_video.py "基于样片保持动作节奏，提升细节和质感" \
  --video ~/Desktop/draft.mp4
```

### 4. Inspect an existing task only

```bash
python3 scripts/generate_video.py --task-id <task_id> --wait false
```

### 5. Submit only, do not wait

```bash
python3 scripts/generate_video.py "产品广告短片，金属质感，高级商业风" \
  --ratio 16:9 \
  --wait false
```

### 6. Use raw content JSON when the target model needs a custom body

```bash
python3 scripts/generate_video.py --content-json '[
  {"type":"text","text":"一只橘猫坐在窗边看雨"},
  {"type":"image_url","image_url":{"url":"https://example.com/ref.png"}}
]'
```

## Local media support

For `--image` and `--video`:

- local file path → converted to `data:` URL automatically
- `https://...` URL → sent as-is
- `data:...` URL → sent as-is

This makes local reference media usable without manual upload steps.

## Key options

- `--ratio 16:9|9:16|1:1|adaptive`
- `--duration <seconds>`
- `--frames <count>`
- `--seed <int>`
- `--resolution <value>`
- `--camera-fixed true|false`
- `--watermark true|false`
- `--callback-url <https-url>`
- `--poll-interval <seconds>`
- `--timeout <seconds>`
- `--download-results true|false`
- `--download-dir <path>`
- `--print-request`

## Execution checklist

1. Confirm whether the user wants text-to-video, image-to-video, or draft-video refinement.
2. Choose prompt-first mode by default; use `--content-json` only when the API shape must be customized.
3. Pass local reference media directly with `--image` or `--video`.
4. Prefer `--duration` for whole-second clips and `--frames` only when finer control is required.
5. Poll by default so the final answer includes actual output URLs or downloaded files.
6. Mention the saved file paths when downloads are enabled.
7. If the API returns an unexpected structure, surface the raw JSON instead of guessing.

## Troubleshooting

- Missing key → set `VOLCENGINE_API_KEY`
- Missing model → set `VOLCENGINE_VIDEO_MODEL`
- Missing endpoint → set `VOLCENGINE_VIDEO_ENDPOINT`
- 401/403 → key invalid or missing permission
- 404 → endpoint wrong or region mismatch
- 400 → unsupported model/parameter combination
- Task remains queued too long → check quota, rate limit, or model availability
- No obvious video URL in response → inspect `raw`

## References

- `references/sources.md`
- `references/api-notes.md`
