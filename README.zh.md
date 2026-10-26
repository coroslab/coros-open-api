[English](README.md)｜中文｜[快速开始](api-reference.zh-CN.md#quick-start)｜[接口文档](api-reference.zh-CN.md)｜[常见问题](api-reference.zh-CN.md#faq)

# COROS 开放平台接口文档

文档版本：V2.1.1

面向接入 COROS 的合作伙伴，介绍用户授权、运动与日常数据查询、数据推送、训练计划、路线和运动文件上传。

## 阅读入口

| 语言 | 接口文档 | 接入指引 |
| --- | --- | --- |
| 简体中文 | [V2.1.1](api-reference.zh-CN.md) | [快速开始](api-reference.zh-CN.md#quick-start) · [常见问题](api-reference.zh-CN.md#faq) |
| English | [V2.1.1](api-reference.md) | [Quick start](api-reference.md#quick-start) · [FAQ](api-reference.md#faq) |

<a id="access-and-support"></a>

## 接入与反馈

接入需要先向 COROS 对接人申请并取得应用凭证。文档公开不代表接口无需申请即可使用。业务支持请联系 COROS 对接人。文档纠错可通过仓库的 [Issues 页面](https://github.com/coroslab/coros-open-api/issues)提交；该页面仅在你有仓库访问权限且仓库已开启 Issues 时可用。若页面不可用，请将问题交给 COROS 对接人转交维护者。

反馈时请提供文档版本、接口、环境、发生时间和去除敏感信息的错误响应。请勿公开密钥、授权凭证、签名下载地址或用户数据。

尚无 COROS 对接人时，可在上述 Issues 页面可用的情况下询问接入申请渠道；请勿在公开问题中提交申请凭证或用户资料。

## 文档纠错

提交文档纠错时，请注明文档语言、章节、原文和建议修改内容。也可通过仓库的 Pull requests 提交修改，并同步更新受影响的中英文说明。涉及接口行为、版本号或历史日期的修改，请提供可核实的依据；无法确认时先提交问题说明。

## 版本

当前文档版本为 V2.1.1；历史变化见各语言文档的修订记录。2026-10-26 的整理修复了示例、链接和语言差异，并增加接入指引，不代表接口行为变更。

## 环境

| 环境 | 地址 |
| --- | --- |
| 测试 | `https://opentest.coros.com` |
| 正式 | `https://open.coros.com` |

请先阅读对应语言的快速开始，完成接入申请和用户授权，再按各接口说明调用。示例中的凭证、日期和文件地址需要替换；`example.com` 下的文件地址不能用于实际下载。

## 文档范围

本仓库提供接入参考文档，不包含 SDK 或可直接运行的应用。文档版本为 V2.1.1，固定文件名便于收藏和引用；版本变化以文档内修订记录为准。
