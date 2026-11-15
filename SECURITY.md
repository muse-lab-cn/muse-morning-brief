# Security / 安全说明

## What this project is / 这个项目是什么

A skill file (documentation) for a daily morning briefing: weather,
read-only vehicle status, health, and reminders. It contains no code
that runs on its own and no vehicle-control capability.

一份每日早报的 skill 文件（文档），包含天气、只读车辆状态、健康和
提醒。它本身不是能独立运行的代码，也没有任何控车能力。

## Rules / 规矩

- **No secrets in this repo.** No private keys, tokens, client
  secrets, authorization codes, VINs, license plates, live
  locations, account details, or health records — in files, commit
  history, issues, or screenshots. Credentials live only in the
  user's own secure storage and job configuration.
  **仓库里不放任何密钥。**私钥、token、client secret、授权码、
  VIN、车牌、实时位置、账号信息、健康记录——文件、commit 历史、
  issue、截图里都不放。凭据只存在用户自己的安全存储和任务配置里。
- **Vehicle is read-only, always.** This skill never sends a vehicle
  command. Waking a sleeping vehicle to read its status is allowed
  at most once per briefing, only with the user's explicit
  authorization; a failed wake is never retried.
  **车辆永远只读。**这个 skill 绝不发车辆命令。唤醒睡着的车来读数，
  每次早报最多一次，而且必须用户明确授权；唤醒失败不重试。
- **Health data stays with the user.** Sleep and activity data are
  read from the user's own authorized health source and reported in
  their own chat. Nothing is uploaded, shared, or stored in this repo.
  **健康数据只留在用户手里。**睡眠和活动数据从用户自己授权的健康
  来源读取，在用户自己的聊天里播报，不上传、不分享、不存进仓库。
- **No continuous tracking.** One scheduled read per morning is not
  real-time monitoring. Urgent vehicle events still belong to the
  manufacturer's own app.
  **不做持续追踪。**每天早上定时查一次不是实时监控，紧急的车辆
  事件还是以厂家自己的 App 为准。

## If something leaks / 万一泄露了

1. Rotate or revoke the affected credential (token, key, or
   authorization) first.
2. Remove the leaked material from the repository history.
3. Re-check that no other copy (fork, screenshot, issue) still
   carries it.

## Disclaimer / 免责

This project is provided as-is, without warranty of any kind. It is
a personal, non-commercial exploration, not affiliated with Tesla,
Apple, Meta, or any vehicle or health brand mentioned.

本项目按现状提供，不作任何保证。属于个人非商业探索，与特斯拉、
苹果、Meta 及文中提到的任何车辆、健康品牌官方无关。
