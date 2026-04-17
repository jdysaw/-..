<!--
 * 严肃声明：
 * 开源版本请务必保留此注释头信息，若删除我方将保留所有法律责任追究！
 * Copyright (c) 2020 陈尼克 all rights reserved.
-->

<template>
  <div class="product-detail">
    <s-header :name="'商品详情'"></s-header>
    <div class="detail-content">
      <div class="detail-swipe-wrap">
        <van-swipe class="my-swipe" indicator-color="#1baeae">
          <van-swipe-item v-for="(item, index) in state.detail.goodsCarouselList" :key="index">
            <img :src="item" alt="">
          </van-swipe-item>
        </van-swipe>
      </div>
      <div class="product-info">
        <div class="product-title">
          {{ state.detail.goodsName || '' }}
        </div>
        <div class="product-desc">免邮费 顺丰快递</div>
        <div class="product-price">
          <span>¥{{ state.detail.sellingPrice || '' }}</span>
        </div>
      </div>
      <div class="product-intro">
        <ul>
          <li class="active">概述</li>
          <li>参数</li>
          <li>安装服务</li>
          <li>常见问题</li>
        </ul>
        <!-- 核心内容注入点：优先渲染 renderContent -->
        <div class="product-content" v-html="renderContent"></div>
      </div>
    </div>
    <van-action-bar>
      <van-action-bar-icon icon="chat-o" text="客服" />
      <van-action-bar-icon icon="cart-o" :badge="!cart.count ? '' : cart.count" @click="goTo()" text="购物车" />
      <van-action-bar-button type="warning" @click="handleAddCart" text="加入购物车" />
      <van-action-bar-button type="danger" @click="goToCart" text="立即购买" />
    </van-action-bar>
    <van-back-top target=".detail-content" />
  </div>
</template>

<script setup>
import { reactive, onMounted, nextTick, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useCartStore } from '@/stores/cart'
import { getDetail } from '@/service/good'
import { addCart } from '@/service/cart'
import sHeader from '@/components/SimpleHeader.vue'
import { showSuccessToast } from 'vant'
import { prefix } from '@/common/js/utils'

const route = useRoute()
const router = useRouter()
const cart = useCartStore()

const state = reactive({
  detail: {
    goodsCarouselList: []
  }
})

// 计算属性：渲染最终内容（带自动回退逻辑）
const renderContent = computed(() => {
  const content = state.detail.goodsDetailContent || ''
  // 检查是否需要加载演示模板（如果原生内容为空或仅为加载中占位）
  if (!content || !content.includes('<img') || content.includes('加载中')) {
    return defaultContent.value
  }
  return content
})

// 计算属性：华为高级视觉模版（纯内嵌注入）
const defaultContent = computed(() => {
  const name = state.detail.goodsName || ''
  return `
    <div class="hw-product-detail-template">
      <!-- 6大核心理由网格 -->
      <h2 class="hw-section-title">选择 ${name} 的 6 大理由</h2>
      <div class="hw-reasons-grid">
        <div class="hw-reason-card"><span class="icon">🖥️</span><h3>超感光视野</h3><p>色彩精准，原彩显示技术，呈现自然本色。</p></div>
        <div class="hw-reason-card"><span class="icon">🚀</span><h3>性能爆发</h3><p>强劲核心架构，多任务处理迅捷如雷。</p></div>
        <div class="hw-reason-card"><span class="icon">🔋</span><h3>长效续航</h3><p>先进电池管理系统，为您提供全天候动力。</p></div>
        <div class="hw-reason-card"><span class="icon">🛡️</span><h3>坚韧可靠</h3><p>通过多重严苛测试，保证每一处细节经久耐用。</p></div>
        <div class="hw-reason-card"><span class="icon">⚖️</span><h3>极致轻薄</h3><p>航空级材质打造，极简主义，触手轻盈。</p></div>
        <div class="hw-reason-card"><span class="icon">🎵</span><h3>沉浸音效</h3><p>双扬声器立体声场，带来影院级视听盛宴。</p></div>
      </div>

      <!-- 沉浸式黑色技术亮点 -->
      <div class="hw-tech-black-block">
        <h2>全方位技术革新</h2>
        <p>通过深度的软硬件协同优化，${name} 将科技与美学完美融合。无论是极致的输入反馈还是澎湃的算力，都为您每一次操作注入灵感。</p>
      </div>

      <!-- 核心参数区域 -->
      <h2 class="hw-section-title">详细参数矩阵</h2>
      <div class="hw-params-table">
        <div class="hw-p-row"><span>显示系统</span><span>旗舰级全向视野</span></div>
        <div class="hw-p-row"><span>影像参数</span><span>超感光多摄阵列</span></div>
        <div class="hw-p-row"><span>充电方案</span><span>超级快充 + 智能守护</span></div>
        <div class="hw-p-row"><span>机身重量</span><span>约 1.4kg (以实测为准)</span></div>
      </div>

      <!-- 美学文案区 -->
      <div class="hw-aesthetic-area">
        <h2>轻盈于形 · 强劲于心</h2>
        <p class="hw-note">*实际续航依具体使用情况及环境而定。无线充电套装需另行购买。</p>
      </div>

      <!-- 底部服务保障横条 -->
      <div class="hw-service-footer">
        <div class="hw-s-item"><span>🛡️</span>正品保障</div>
        <div class="hw-s-item"><span>🚀</span>顺丰包邮</div>
        <div class="hw-s-item"><span>📅</span>售后无忧</div>
      </div>
    </div>
  `
})

onMounted(async () => {
  const { id } = route.params
  const { data } = await getDetail(id)
  data.goodsCarouselList = data.goodsCarouselList.map(i => prefix(i))
  state.detail = data
  cart.updateCart()
})

nextTick(() => {
  const content = document.querySelector('.detail-content')
  if (content) content.scrollTop = 0
})

const goTo = () => {
  router.push({ path: '/cart' })
}

const handleAddCart = async () => {
  const { resultCode } = await addCart({ goodsCount: 1, goodsId: state.detail.goodsId })
  if (resultCode == 200 ) showSuccessToast('添加成功')
  cart.updateCart()
}

const goToCart = async () => {
  await addCart({ goodsCount: 1, goodsId: state.detail.goodsId })
  cart.updateCart()
  router.push({ path: '/cart' })
}
</script>

<style lang="less">
  @import '../common/style/mixin';
  .product-detail {
    .detail-content {
      height: calc(100vh - 50px);
      overflow: hidden;
      overflow-y: auto;
      .detail-swipe-wrap {
        .my-swipe .van-swipe-item {
          img {
            width: 100%;
          }
        }
      }
      .product-info {
        padding: 0 15px;
        .product-title {
          font-size: 20px;
          text-align: left;
          color: #333;
          font-weight: 500;
          padding-top: 10px;
        }
        .product-desc {
          font-size: 14px;
          text-align: left;
          color: #999;
          padding: 5px 0;
        }
        .product-price {
          padding: 5px 0;
          span {
            color: #F63515;
            font-size: 24px;
            font-weight: bold;
          }
        }
      }
      .product-intro {
        width: 100%;
        margin-top: 20px;
        ul {
          display: flex;
          width: 100%;
          border-bottom: 1px solid #eee;
          li {
            flex: 1;
            padding: 15px 0;
            text-align: center;
            font-size: 15px;
            color: #666;
            &.active {
              color: #1baeae;
              font-weight: bold;
              border-bottom: 2px solid #1baeae;
            }
          }
        }
        .product-content {
          padding: 0; 
          
          /* 华为官网级视觉样式 */
          .hw-product-detail-template {
            color: #333;
            background: #fff;
            .hw-section-title {
              text-align: center;
              font-size: 22px;
              padding: 40px 15px 25px;
              font-weight: 600;
              letter-spacing: 1px;
            }
            .hw-reasons-grid {
              display: grid;
              grid-template-columns: 1fr 1fr;
              gap: 15px;
              padding: 0 20px 40px;
              .hw-reason-card {
                background: #fdfdfd;
                border: 1px solid #f2f2f2;
                padding: 20px 10px;
                border-radius: 16px;
                text-align: center;
                transition: transform 0.2s;
                .icon { font-size: 28px; margin-bottom: 10px; display: block; }
                h3 { font-size: 15px; margin-bottom: 6px; font-weight: bold; color: #111; }
                p { font-size: 11px; color: #999; line-height: 1.5; padding: 0 5px; }
              }
            }
            .hw-tech-black-block {
              background: #000;
              color: #fff;
              padding: 70px 25px;
              text-align: center;
              margin: 30px 0;
              h2 { font-size: 28px; margin-bottom: 15px; background: linear-gradient(to bottom, #fff, #999); -webkit-background-clip: text; -webkit-text-fill-color: transparent; font-weight: bold; }
              p { font-size: 14px; line-height: 1.8; color: #ccc; max-width: 90%; margin: 0 auto; }
            }
            .hw-params-table {
              margin: 0 20px 40px;
              border: 1px solid #eee;
              border-radius: 14px;
              overflow: hidden;
              .hw-p-row {
                display: flex;
                justify-content: space-between;
                padding: 18px 20px;
                border-bottom: 1px solid #f8f8f8;
                font-size: 14px;
                &:last-child { border-bottom: none; }
                span:first-child { color: #888; font-weight: normal; }
                span:last-child { font-weight: 600; color: #333; }
              }
            }
            .hw-aesthetic-area {
              text-align: center;
              padding: 60px 20px;
              h2 { font-style: italic; font-size: 30px; color: #111; margin-bottom: 10px; font-weight: 300; }
              .hw-note { font-size: 11px; color: #bbb; margin-top: 30px; letter-spacing: 0.5px; }
            }
            .hw-service-footer {
              display: flex;
              justify-content: space-around;
              padding: 30px 10px;
              background: #fafafa;
              border-top: 1px solid #eee;
              .hw-s-item { font-size: 13px; color: #444; font-weight: 500; span { margin-right: 5px; font-size: 18px; } }
            }
          }
        }
      }
    }
    .van-action-bar-button--warning {
      background: linear-gradient(to right,#6bd8d8, @primary)
    }
    .van-action-bar-button--danger {
      background: linear-gradient(to right, #0dc3c3, #098888)
    }
  }
</style>
