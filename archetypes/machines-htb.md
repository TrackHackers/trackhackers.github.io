---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: ["HTB", "Write-up"]
categories: ["Machines"]
htb_link: https://app.hackthebox.com/machines/{{ .Name }}
level: easy
os: win
---

## Machine Summary

| MachineName | {{ .Params.title }}    |
| ----------- | ---------------------- |
| HTB Link    | {{ .Params.htb_link }} |
| OS          | {{ .Params.os }}       |
| Level       | {{ .Params.level }}    |
| Seasonal    | No                     |

## Introduction

<!-- Brief intro here -->

## Enumeration

<!-- Describe enumeration steps -->

## Exploitation

<!-- Describe the vulnerabilities you found -->

## Privilege Escalation

<!-- Describe your steps for privilege escalation -->

## Conclusion

<!-- Wrap up -->
