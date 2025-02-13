/*
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing,
 * software distributed under the License is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 * KIND, either express or implied.  See the License for the
 * specific language governing permissions and limitations
 * under the License.
 */
<template>
  <main class="page">
    <slot name="top"></slot>
    <div class="content-wrapper" :class="{ 'have-rightmenu': showRightMenu }">
      <RightMenu v-if="showRightMenu" />
      <Content class="content" />
    </div>
    <span id="doc-version" style="display: none;">{{ docVersion }}</span>
    <!-- <footer class="page-edit">
      <blockquote>
        <p>
          Anything unclear or missing?
          <a :href="commentLink" target="_blank" rel="noopener noreferrer">
            Leave a comment
          </a>
          <OutboundLink/> or <a
            :href="editLink"
            target="_blank"
            rel="noopener noreferrer"
          >{{ editLinkText }}</a>
          <OutboundLink/>
        </p>
        <small>
          <span class="prefix">{{ lastUpdatedText }}: </span>
          <span class="time">{{ lastUpdated }}</span>
        </small>

      </blockquote>
    </footer> -->

    <div class="page-nav" v-if="prev || next">
      <p class="inner">
        <span
          v-if="prev"
          class="prev"
        >
          ←
          <router-link
            v-if="prev"
            class="prev"
            :to="prev.path"
          >
            {{ prev.title || prev.path }}
          </router-link>
        </span>

        <span
          v-if="next"
          class="next"
        >
          <router-link
            v-if="next"
            :to="next.path"
          >
            {{ next.title || next.path }}
          </router-link>
          →
        </span>
      </p>
    </div>

    <p style="text-align: center; color: #909399; font-size: 16px; margin: 0 30px;">
      <a
        :href="editLink"
        target="_blank"
        rel="noopener noreferrer"
      >{{ editLinkText }}</a>
      <OutboundLink />
    </p>
    <p style="text-align: center; color: #909399; font-size: 12px; margin: 0 30px;">Copyright © 2022 The Apache Software Foundation.<br>
      Apache and the Apache feather logo are trademarks of The Apache Software Foundation</p>
    <p style="text-align: center; margin-top: 10px; color: #909399; font-size: 12px; margin: 0 30px;">
      <strong>Have a question?</strong> Connect with us on QQ, WeChat, or Slack. <a href="https://github.com/apache/iotdb/issues/1995">Join the community</a> now.</p>
    <p style="text-align: center; margin-top: 10px; color: #909399; font-size: 12px; margin: 0 30px;">
      We use <a href="https://analytics.google.com">Google Analytics</a> to collect anonymous, aggregated usage information.
    </p>
  </main>
</template>

<script>
import RightMenu from './RightMenu.vue';
import { resolvePage, outboundRE, endingSlashRE } from '../util';

export default {
  props: ['sidebarItems'],
  components: { RightMenu },
  computed: {
    showRightMenu() {
      const { $frontmatter, $themeConfig, $page } = this;
      const { sidebar } = $frontmatter;
      return (
        $themeConfig.rightMenuBar !== false
        && $page.headers
        && ($frontmatter && sidebar && sidebar !== false) !== false
      );
    },
    lastUpdated() {
      return this.$page.lastUpdated;
    },

    lastUpdatedText() {
      if (typeof this.$themeLocaleConfig.lastUpdated === 'string') {
        return this.$themeLocaleConfig.lastUpdated;
      }
      if (typeof this.$site.themeConfig.lastUpdated === 'string') {
        return this.$site.themeConfig.lastUpdated;
      }
      return 'Last Updated';
    },

    prev() {
      const { prev } = this.$page.frontmatter;
      if (prev === false) {

      } else if (prev) {
        return resolvePage(this.$site.pages, prev, this.$route.path);
      } else {
        return resolvePrev(this.$page, this.sidebarItems);
      }
    },

    next() {
      const { next } = this.$page.frontmatter;
      if (next === false) {

      } else if (next) {
        return resolvePage(this.$site.pages, next, this.$route.path);
      } else {
        return resolveNext(this.$page, this.sidebarItems);
      }
    },

    editLink() {
      if (this.$page.frontmatter.editLink === false) {
        return;
      }
      const {
        repo,
        editLinks,
        docsDir = '',
        docsBranch = 'master',
        docsRepo = repo,
      } = this.$site.themeConfig;

      if (docsRepo && editLinks && this.$page.relativePath) {
        return this.createEditLink(repo, docsRepo, docsDir, docsBranch, this.$page.relativePath);
      }
    },

    commentLink() {
      const {
        repo,
        docsRepo = repo,
      } = this.$site.themeConfig;

      return `${outboundRE.test(docsRepo)
        ? docsRepo
        : `https://github.com/${docsRepo}`
      }/issues/new?title=Comment: ${this.$page.title} (${this.$page.relativePath})`;
    },

    editLinkText() {
      return (
        this.$themeLocaleConfig.editLinkText
        || this.$site.themeConfig.editLinkText
        || 'Edit this page'
      );
    },
    docVersion() {
      return this.getBranch('master', this.$page.relativePath);
    },
  },

  methods: {
    getBranch(branch = 'master', path) {
      if (path.indexOf('UserGuide/Master') > -1 || path.indexOf('UserGuide') === -1) {
        return branch;
      }
      const branchRex = /UserGuide\/V(\d+\.\d+\.x)/;
      if (branchRex.test(path)) {
        const tag = branchRex.exec(path)[1];
        return `rel/${tag.replace('.x', '')}`;
      }
      return branch;
    },
    getPath(path) {
      if (path.indexOf('UserGuide/Master') > -1) {
        return path.replace('UserGuide/Master', 'UserGuide');
      }
      const branchRex = /UserGuide\/V(\d+\.\d+\.x)/;
      if (branchRex.test(path)) {
        const tag = branchRex.exec(path)[1];
        return path.replace(`UserGuide/V${tag}`, 'UserGuide');
      }
      return path;
    },
    createEditLink(repo, docsRepo, docsDir, docsBranch, path) {
      const bitbucket = /bitbucket.org/;
      if (bitbucket.test(repo)) {
        const base = outboundRE.test(docsRepo)
          ? docsRepo
          : repo;
        return (
          `${base.replace(endingSlashRE, '')
          }/src`
           + `/${docsBranch}/${
             docsDir ? `${docsDir.replace(endingSlashRE, '')}/` : ''
           }${path
           }?mode=edit&spa=0&at=${docsBranch}&fileviewer=file-view-default`
        );
      }

      const base = outboundRE.test(docsRepo)
        ? docsRepo
        : `https://github.com/${docsRepo}`;
      return (
        `${base.replace(endingSlashRE, '')
        }/edit`
        + `/${this.getBranch(docsBranch, path)}/${
          docsDir ? `${docsDir.replace(endingSlashRE, '')}/` : ''
        }${this.getPath(path)}`
      );
    },
  },
};

function resolvePrev(page, items) {
  return find(page, items, -1);
}

function resolveNext(page, items) {
  return find(page, items, 1);
}

function find(page, items, offset) {
  const res = [];
  flatten(items, res);
  for (let i = 0; i < res.length; i++) {
    const cur = res[i];
    if (cur.type === 'page' && cur.path === decodeURIComponent(page.path)) {
      return res[i + offset];
    }
  }
}

function flatten(items, res) {
  for (let i = 0, l = items.length; i < l; i++) {
    if (items[i].type === 'group') {
      flatten(items[i].children || [], res);
    } else {
      res.push(items[i]);
    }
  }
}

</script>

<style lang="stylus">
@require '../styles/wrapper.styl'

.page
  padding-bottom 2rem
  display block

.page-edit
  @extend $wrapper
  padding-top 0
  padding-bottom 1rem
  overflow auto
  .edit-link
    display inline-block
    a
      color lighten($textColor, 25%)
      margin-right 0.25rem
  .last-updated
    float right
    font-size 0.9em
    .prefix
      font-weight 500
      color lighten($textColor, 25%)
    .time
      font-weight 400
      color #aaa

.page-nav
  @extend $wrapper
  padding-top 1rem
  padding-bottom 0
  .inner
    min-height 2rem
    margin-top 0
    border-top 1px solid $borderColor
    padding-top 1rem
    overflow auto // clear float
  .next
    float right

@media (max-width: $MQMobile)
  .page-edit
    .edit-link
      margin-bottom .5rem
    .last-updated
      font-size .8em
      float none
      text-align left
.page:not(.have-rightmenu)
  .content
    margin-right auto
.page:has(.have-rightmenu)
  .content
    margin-right 230px
  @media (max-width 1100px)
    .content
      margin-right auto
@media (max-width 1100px)
  .have-rightmenu
    .right-menu-wrapper
      display none

</style>
