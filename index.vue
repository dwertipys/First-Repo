<template>
  <client-only>
    <div class="html2md-container">
      <div class="banner-box">
        <a class="home-link" href="https://www.helloworld.net" target="_blank">
          <img
            class="logo"
            width="100"
            src="/img/logo-white.png"
            alt="home-link"
          />
        </a>
        <a
          class="github-link"
          href="https://github.com/helloworld-Co/html2md"
          target="_blank"
        >
          <img width="36" src="/img/github-white.png" alt="github-link" />
        </a>
      </div>
      <a-tabs type="card">
        <a-tab-pane key="1" tab="url一键转md">
          <div class="main-box">
            <div class="form-item">
              <a-input
                v-model="url"
                size="large"
                placeholder="请输入文章地址"
                @keyup.enter.native="transformUrl"
                @focus="getInputFocus($event)"
              />
              <div>&nbsp;</div>
              <a-button
                size="large"
                icon="refresh"
                :loading="isLoading"
                type="primary"
                @click="transformUrl"
                >一键转换</a-button
              >
              <a-button
                size="large"
                icon="printer"
                type="primary"
                :loading="isLoading"
                ghost
                @click="saveMd('md')"
                >保存为 MD</a-button
              >
            </div>
            <div class="form-item">
              <a-input
                v-model="title"
                size="large"
                placeholder="标题（自动读取）"
                @focus="getInputFocus($event)"
              />
            </div>

            <hw-editor
              ref="editor"
              v-model="md"
              :disabled-menus="['image/upload-image']"
              class="md-editor"
              @save="saveMd('md')"
            />
            <a-icon
              type="copy"
              class="copy-btn"
              title="复制"
              @click="onCopyMd"
            />
          </div>
        </a-tab-pane>
        <a-tab-pane key="2" tab="html转md">
          <div class="copyhtml2md-box">
            <div class="save-btns">
              <p class="tips">
                Tip: 复制 html 内容到左边的文本框，无需操作，自动转换 👏🏻
              </p>
              <a-button
                size="large"
                icon="printer"
                type="primary"
                :loading="isLoading"
                ghost
                @click="saveMd('copyMd')"
                >保存为 MD</a-button
              >
            </div>
            <div class="input-box">
              <a-input
                v-model="copyHtml"
                size="large"
                type="textarea"
                autosize
                placeholder="请输入html内容"
                @focus="getInputFocus($event)"
              >
              </a-input>
              <a-textarea
                v-model="copyMd"
                size="large"
                type="textarea"
                autosize
                placeholder="这里是转换后的md"
                @focus="getInputFocus($event)"
              />
            </div>
          </div>
        </a-tab-pane>
      </a-tabs>
    </div>
  </client-only>
</template>

<script>
import copy from 'copy-to-clipboard'
import TurndownService from 'turndown'
import { gfm, tables, strikethrough } from 'turndown-plugin-gfm'

export default {
  layout: 'h5',
  name: 'Html2mdBox',
  data() {
    return {
      md: '```\nHello world!\n```',
      html: '<pre>Hello world!</pre>',
      url: '',
      isLoading: false,
      title: '',
      copyHtml: '<pre>Hello world!</pre>',
      logos: [
        {
          logo: '/img/logos/csdn.svg'
        },
        {
          logo: '/img/logos/juejin.svg',
          height: 20
        },
        {
          logo: '/img/logos/jianshu.png',
          height: 18
        },
        {
          logo: '/img/logos/segmentfault.svg',
          height: 20
        },
        {
          logo: '/img/logos/cnblogs.svg',
          height: 20
        },
        {
          logo: '/img/logos/oschina.svg',
          height: 22
        },
        {
          logo: '/img/logos/weixin.jpg',
          height: 20
        }
      ]
    }
  },
  components: {},
  computed: {
    copyMd() {
      return this.html2md(this.copyHtml)
    }
  },
  methods: {
    onCopyMd() {
      copy(this.md)
      this.$message.success('复制成功')
    },
    downLoadFile(url) {
      const a = document.createElement('a')
      a.download = `${Date.now()}.md`
      a.href = url
      a.click()
    },
    saveMd(name) {
      this.isLoading = true
      const params = {
        md: this[name],
        url: window.location.origin
      }
      this.$axios
        .post(`${window.location.origin}/getMdFile`, params)
        .then((res) => {
          this.downLoadFile(res.path)
          this.isLoading = false
        })
        .catch(() => {
          this.isLoading = false
        })
    },
    html2md(str) {
      const turndownService = new TurndownService({ codeBlockStyle: 'fenced' })
      // Use the gfm plugin
      turndownService.use(gfm)

      // Use the table and strikethrough plugins only
      turndownService.use([tables, strikethrough])
      // 自定义配置
      turndownService.addRule('pre2Code', {
        filter: ['pre'],
        replacement(content) {
          const len = content.length
          // 除了pre标签，里面是否还有code标签包裹，有的话去掉首尾的`（针对微信文章）
          const isCode = content[0] === '`' && content[len - 1] === '`'
          const result = isCode ? content.substr(1, len - 2) : content
          return '```\n' + result + '\n```\n'
        }
      })
      const markdown = turndownService.turndown(str)
      return markdown
    },
    getInputFocus(event) {
      event.currentTarget.select()
    },
    transformUrl() {
      if (!this.url) {
        return
      }
      this.isLoading = true
      this.$axios
        .get(`${window.location.origin}/getUrlHtml`, {
          params: { url: this.url }
        })
        .then((res) => {
          this.html = res.html
          this.md = this.html2md(this.html)
          this.title = res.title
          this.isLoading = false
        })
        .catch(() => {
          this.isLoading = false
        })
    }
  }
}
</script>

<style lang="less">
.html2md-container {
  padding-bottom: 10px;
  .banner-box {
    height: 240px;
    margin: auto;
    background: url(/img/html2md-banner.png) no-repeat #113da4;
    background-size: 1920px 240px;
    background-position: center top;
    position: relative;
    .home-link {
      position: absolute;
      top: 23px;
      left: 50%;
      transform: translateX(-600px);
    }
    .github-link {
      position: absolute;
      top: 23px;
      left: 50%;
      transform: translateX(566px);
    }
  }

  .ant-tabs {
    width: 1220px;
    margin: -40px auto 10px;
    position: relative;
    z-index: 1;
    border-radius: 8px;
    overflow: hidden;
    user-select: none;
    background: #fff;
    border: 1px solid #dcdfe6;
    box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.12), 0 0 6px 0 rgba(0, 0, 0, 0.04);
    .ant-tabs-nav-wrap {
      background: #f5f7fa;
    }
    .ant-tabs-content {
      padding: 15px;
    }
    .ant-tabs-tab {
      border-top: none;
      border-left: 1px solid transparent !important;
      border-right: 1px solid transparent !important;
      border-bottom: 1px solid #e8e8e8 !important;
      &.ant-tabs-tab-active {
        border-left-color: #e8e8e8 !important;
        border-right-color: #e8e8e8 !important;
        border-bottom-color: white !important;
      }
    }
    .ant-tabs-bar {
      margin-bottom: 0 !important;
    }
  }
  .main-box {
    width: 100%;
    background: #ffffff;
    margin: 0 auto;
    position: relative;
    .copy-btn {
      position: absolute;
      top: 170px;
      left: 555px;
      z-index: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      width: 24px;
      height: 24px;
      color: #ddd;
      font-size: 14px;
      background-color: #666;
      border-radius: 6px;
      box-shadow: 0 2px 0 0 rgba(0, 0, 0, 0.2);
      cursor: pointer;
      &:hover {
        opacity: 0.9;
      }
    }
    .ant-btn + .ant-btn {
      margin-left: 10px;
    }
    .ant-input__inner {
      background: #fafafa;
      border: 1px solid #e5e5e5;
      border-radius: 2px;
    }
    .form-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 20px;
    }
    .hw-editor {
      height: 630px;
      margin: 0 auto;
      border-radius: 2px;
      box-shadow: none;
      .hw-editor__main {
        border: 1px solid #e5e5e5;
        background: #fafafa;
      }
      .hw-editor__toolbar {
        background: #0fabf5;
      }
      .hw-editor__toolbar-item {
        color: #fff;
      }
      .hw-editor__toolbar-item--active,
      .hw-editor__toolbar-item--active:hover,
      .hw-editor__toolbar-item:hover {
        background: rgba(38, 67, 125, 0.2);
      }
      .hw-textarea-editor textarea {
        background: #fafafa;
      }
      .hw-editor__left-area {
        border: 1px solid #e5e5e5;
        border-right: none;
      }
    }
  }

  .copyhtml2md-box {
    overflow: hidden;
    .save-btns {
      padding-bottom: 20px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      .tips {
        color: #999;
        font-size: 14px;
      }
    }
    .input-box {
      width: 100%;
      height: 630px;
      background: #ffffff;
      margin: 0 auto;
      display: flex;
      justify-content: space-between;
      align-items: flex-start;
      textarea.ant-input {
        flex: 1;
        height: 100% !important;
        border: 1px solid #e5e5e5;
        background: #fafafa;
        border-radius: 2px;
        & + textarea.ant-input {
          margin-left: 10px;
        }
      }
    }
  }
}

@media screen and (max-width: 1200px) {
  .html2md-container {
    .ant-tabs {
      width: 100%;
    }
  }
}
</style>
