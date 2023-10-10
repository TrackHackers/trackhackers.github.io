---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: ["HTB", "Seasonal", "Write-up"]
categories: ["Seasonal Machines"]
htb_link: https://app.hackthebox.com/competitive/3/overview
level: easy
os: win
---

## Machine Summary

| MachineName | {{ .Params.title }}    |
| ----------- | ---------------------- |
| HTB Link    | {{ .Params.htb_link }} |
| OS          | {{ .Params.os }}       |
| Level       | {{ .Params.level }}    |
| Seasonal    | Yes                    |

## Introduction

<!-- Brief intro with a seasonal twist maybe? -->

## Special Enumeration

<!-- Describe any seasonal-specific enumeration steps -->

## Seasonal Exploitation

<!-- Describe the seasonal vulnerabilities you found -->

## Seasonal Privilege Escalation

<!-- Describe your steps for seasonal privilege escalation -->

## Conclusion

<!-- Wrap up with seasonal insights -->
