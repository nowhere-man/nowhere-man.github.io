---
layout: default
title: 首页
---

<div class="bg-blue-600 text-white py-12 px-4 text-center">
    <div class="container mx-auto">
        <h1 class="text-4xl md:text-5xl font-bold mb-4">欢迎来到我的技术分享博客</h1>
        <p class="text-lg md:text-xl mb-6">这里记录着我对前端、后端、云计算等技术的学习与思考。</p>
        <div class="max-w-xl mx-auto">
            <form id="search-form" class="relative">
                <input type="text" id="search-input" placeholder="搜索文章标题..."
                    class="w-full py-3 px-6 rounded-full text-gray-800 focus:outline-none focus:ring-2 focus:ring-blue-400">
                <button type="submit"
                    class="absolute right-0 top-0 mt-2 mr-2 bg-blue-500 text-white rounded-full p-2 hover:bg-blue-400 transition-colors duration-300">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                        xmlns="http://www.w3.org/2000/svg">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                    </svg>
                </button>
            </form>
        </div>
    </div>
</div>

<div class="container mx-auto mt-8 px-4 grid grid-cols-1 md:grid-cols-3 gap-8" id="main-content">
    <div class="col-span-1 md:col-span-2">
        <div id="search-results"></div>
        <div id="post-list">
            <h2 class="text-2xl font-bold text-gray-800 mb-4">最新文章</h2>
            {% for post in site.posts limit:50 %}
                {% include post_card.html post=post %}
            {% endfor %}
        </div>
    </div>

    <aside class="col-span-1 sidebar">
        {% include sidebar.html %}
    </aside>
</div>
