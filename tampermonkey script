// ==UserScript==
// @name         YNAB Sidebar TOTAL Row (Cash + Credit, wide padding)
// @match        *://app.ynab.com/*
// @run-at       document-end
// @grant        none
// ==/UserScript==
(function () {
  'use strict';

  const qs  = (s, r=document) => r.querySelector(s);
  const text = (el) => (el?.textContent || '').trim();

  function parseSigned(el) {
    if (!el) return 0;
    const num = parseFloat(text(el).replace(/[^0-9.]/g, '')) || 0;
    return el.classList.contains('negative') ? -num : num;
  }

  function detectSymbol(el) {
    if (!el) return '$';
    const m = text(el).match(/[$€£¥₩₽₹₺₫₪₱₦₴₭฿₡₲₸₵]/);
    return m ? m[0] : '$';
  }

  function fmt(n, symbol) {
    const sign = n < 0 ? '−' : '';
    const abs = Math.abs(n);
    return `${sign}${symbol}${abs.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
  }

  function computeTotals() {
    const cashEl   = qs('.nav-account.cash   .nav-account-block .nav-account-value .currency');
    const creditEl = qs('.nav-account.credit .nav-account-block .nav-account-value .currency');

    const symbol = detectSymbol(cashEl || creditEl);
    const cash   = parseSigned(cashEl);
    const credit = parseSigned(creditEl);
    const total  = cash + credit;

    return { total, symbol };
  }

  function ensureTotalRow() {
    const container = qs('nav.sidebar .nav-accounts') || qs('nav.ynab-u.sidebar .nav-accounts') || qs('.nav-accounts');
    if (!container) return null;

    let row = qs('#ynab-total-row', container);
    if (!row) {
      row = document.createElement('div');
      row.id = 'ynab-total-row';
      row.style.cssText = `
        display:flex;
        justify-content:space-between;
        align-items:center;
        padding: 6px 50px;        /* 36px left/right padding */
        margin: 6px 0 10px 0;
        font-weight: 500;
        font-family: Inter, system-ui, sans-serif;
        border-radius: 8px;
        background: var(--v-secondary-surface, rgba(255,255,255,0.06));
        border: 1px solid rgba(127,127,127,0.25);
      `;

      const label = document.createElement('div');
      label.textContent = 'TOTAL';
      label.style.opacity = '0.9';

      const value = document.createElement('div');
      value.id = 'ynab-total-amount';
      value.style.cssText = 'font-variant-numeric: tabular-nums; white-space: nowrap;';

      row.appendChild(label);
      row.appendChild(value);

      container.insertBefore(row, container.firstChild);
    }
    return row;
  }

  function render() {
    const row = ensureTotalRow();
    if (!row) return;

    const { total, symbol } = computeTotals();
    const value = qs('#ynab-total-amount', row);
    if (!value) return;

    value.textContent = fmt(total, symbol);
    value.style.color = total < 0 ? '#ff6b6b' : '';
  }

  const mo = new MutationObserver(() => {
    clearTimeout(render._t);
    render._t = setTimeout(render, 150);
  });

  function arm() {
    const sidebar = qs('nav.sidebar') || qs('nav.ynab-u.sidebar') || qs('nav[role="navigation"]');
    if (!sidebar) return false;
    mo.observe(sidebar, { childList: true, subtree: true, characterData: true });
    render();
    return true;
  }

  const boot = setInterval(() => { if (arm()) clearInterval(boot); }, 600);
  setInterval(render, 2000);
})();
