<template>
    <div class="pdf-laypage-wrapper">
        <div ref="wrap"
            :style="{
                height: `${pageHeight}px`
            }">
            <canvas></canvas>
        </div>
        <div class="notice" v-if="show">
            {{ notice }}
        </div>
        <div class="pdf-laypage-tools">
            <button class="pdf-laypage-btn" @click="prevPage">上一页</button>
            <span class="pdf-laypage-page">第 {{ currentPage }} 页 / 共 {{ docPages }} 页</span>
            <button class="pdf-laypage-btn" @click="nextPage">下一页</button>
        </div>
    </div>
</template>

<script>
let scriptPromise = null
export function loadPdfJS() {
    if (scriptPromise) {
        return scriptPromise
    }
    scriptPromise = new Promise((resolve) => {
        var el = document.createElement("script")
        el.src = '/pdfjs-dist/build/pdf.min.js'
        el.onload = resolve
        var s = document.getElementsByTagName('script')[0]
        s.parentNode.insertBefore(el, s)
    })
    return scriptPromise
}
export default {
    props: {
        url: {
            type: String,
            required: true
        },
    },
    data() {
        return {
            doc: null,
            docPages: 0,
            currentPage: 1,
            pageHeight: 0,
            show: false,
            notice: '',
        }
    },
    watch: {
        url: {
            immediate: true,
            handler() {
                loadPdfJS().then(() => {
                    this.getPDFFile()
                })
            }
        }
    },
    methods: {
        getPDFFile() {
            if (!this.url) return
            pdfjsLib.getDocument(this.url).then(pdf => {
                this.doc = pdf
                this.docPages = pdf.numPages
                this.$nextTick(() => {
                    this.docPages && this.renderPage(this.currentPage)
                })
            })
        },
        // 渲染page
        renderPage(pageNo) {
            this.doc.getPage(pageNo).then(page => {
                let wrap = this.$refs.wrap
                if (!wrap) return
                let canvas = wrap.querySelector('canvas')
                let ctx = canvas.getContext('2d')
                let dpr = window.devicePixelRatio || 1
                let bsr = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio
                    || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1
                let ratio = dpr / bsr
                let rect = wrap.getBoundingClientRect()
                let viewport = page.getViewport(1)
                let width = rect.width
                let height = width / viewport.width * viewport.height
                canvas.style.width = `${width}px`
                canvas.style.height = `${height}px`
                this.pageHeight = height
                canvas.height = height * ratio
                canvas.width = width * ratio
                ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
                page.render({
                    canvasContext: ctx,
                    viewport: page.getViewport(width / viewport.width)
                })
                canvas.__rendered = true
            })
        },
        prevPage() {
            this.currentPage = this.currentPage <= 1 ? 1 : this.currentPage - 1
            this.renderPage(this.currentPage)
        },
        nextPage() {
            this.currentPage = this.currentPage >= this.docPages ? this.docPages : this.currentPage + 1
            this.renderPage(this.currentPage)
        },
    }
}
</script>

<style scoped lang="less">
.pdf-laypage-wrapper {

    .pdf-laypage-tools {
        width: 100%;
        height: 40px;
        z-index: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        padding: 0 20px 0 0;
        font-size: 14px;
    }

    .pdf-laypage-btn {
        display: flex;
        align-items: center;
        padding: 2px 8px;
        background: #191931;
        border-radius: 30px;
        border: none;

        font-weight: 500;
        color: #ffffff;
        cursor: pointer;
    }

    .pdf-laypage-page {
        display: inline-block;
        margin: 0 10px;
    }

}
</style>