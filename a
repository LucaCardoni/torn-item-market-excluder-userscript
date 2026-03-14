// ==UserScript==
// @name         Torn Item Market Excluder
// @namespace    https://greasyfork.org/it/users/1579951-lucacardoni
// @version      1.1.0
// @description  Auto-adjusts sell quantities on the Torn Item Market. Add items to the "Exclusions" panel (one per line). Use "Item Name" to keep everything, or "Item Name:5" to keep 5 and sell the rest.
// @author       Luca
// @match        https://www.torn.com/page.php?sid=ItemMarket*
// @grant        none
// @run-at       document-end
// @icon         https://www.torn.com/images/icons/torn_favicon_64x64.png
// ==/UserScript==

(function () {
    "use strict";

    const STORAGE_KEY = "torn_item_market_exclusions";

    function createUI() {
        if (document.getElementById("torn-exclusion-panel")) return;

        const clearBtn = [...document.querySelectorAll("a, button, span")]
            .find(el => el.textContent.trim() === "Clear all");
        if (!clearBtn) return;

        const container = clearBtn.parentElement;
        container.style.position = "relative";

        const btn = document.createElement("button");
        btn.textContent = "Exclusions";
        btn.style.marginLeft = "8px";
        btn.style.fontSize = "12px";
        btn.style.padding = "4px 8px";
        btn.style.cursor = "pointer";
        btn.style.background = "#2b2b2b";
        btn.style.color = "#ccc";
        btn.style.border = "1px solid #444";
        btn.style.borderRadius = "3px";
        container.appendChild(btn);

        const panel = document.createElement("div");
        panel.id = "torn-exclusion-panel";
        panel.style.position = "absolute";
        panel.style.top = "36px";
        panel.style.right = "0";
        panel.style.width = "260px";
        panel.style.background = "#1b1b1b";
        panel.style.border = "1px solid #444";
        panel.style.borderRadius = "4px";
        panel.style.padding = "10px";
        panel.style.boxShadow = "0 4px 12px rgba(0,0,0,0.6)";
        panel.style.display = "none";
        panel.style.zIndex = "9999";
        panel.style.fontFamily = "Arial, sans-serif";

        const title = document.createElement("div");
        title.textContent = "Excluded Items";
        title.style.color = "#aaa";
        title.style.fontSize = "13px";
        title.style.marginBottom = "4px";
        title.style.fontWeight = "bold";

        const help = document.createElement("div");
        help.textContent = "One per line:\n• Item Name (keep everything)\n• Item Name:5 (keep 5)";
        help.style.fontSize = "10px";
        help.style.color = "#777";
        help.style.marginBottom = "8px";
        help.style.whiteSpace = "pre-line";

        const textarea = document.createElement("textarea");
        textarea.style.width = "100%";
        textarea.style.height = "140px";
        textarea.style.background = "#111";
        textarea.style.border = "1px solid #333";
        textarea.style.color = "#ddd";
        textarea.style.fontSize = "12px";
        textarea.style.padding = "6px";
        textarea.style.resize = "vertical";
        textarea.style.fontFamily = "monospace";
        textarea.value = (localStorage.getItem(STORAGE_KEY) || "").trim();

        textarea.addEventListener("input", () => {
            localStorage.setItem(STORAGE_KEY, textarea.value.trim());
        });

        panel.appendChild(title);
        panel.appendChild(help);
        panel.appendChild(textarea);
        container.appendChild(panel);

        btn.onclick = () => {
            panel.style.display = panel.style.display === "none" ? "block" : "none";
        };
    }

    function applyFilter() {
        const raw = localStorage.getItem(STORAGE_KEY) || "";
        const rules = raw.split("\n")
            .map(line => line.trim())
            .filter(Boolean)
            .map(line => {
                const [namePart, keepPart] = line.split(":");
                return {
                    name: namePart.trim(),
                    keep: keepPart ? parseInt(keepPart.trim(), 10) : Infinity
                };
            });

        const rows = document.querySelectorAll(".virtualListing___jl0JE");
        rows.forEach(row => {
            const nameEl = row.querySelector(".name___XmQWk");
            const qtyInput = row.querySelector('.amountInputWrapper___USwSs input.input-money');
            if (!nameEl || !qtyInput) return;

            const itemName = nameEl.textContent.trim();
            const rule = rules.find(r => r.name === itemName);
            if (!rule) return;

            const maxAmount = parseInt(qtyInput.getAttribute("data-money") || "0", 10);
            if (!maxAmount) return;

            const sellAmount = Math.max(0, maxAmount - rule.keep);

            if (qtyInput.value !== String(sellAmount)) {
                qtyInput.value = sellAmount || "";
                qtyInput.dispatchEvent(new Event("input", { bubbles: true }));
                qtyInput.dispatchEvent(new Event("change", { bubbles: true }));
            }
        });
    }

    function watchChanges() {
        let timeout;
        document.addEventListener("input", () => {
            clearTimeout(timeout);
            timeout = setTimeout(applyFilter, 80);
        });
    }

    function waitForPage() {
        const observer = new MutationObserver(() => {
            if (document.querySelector(".virtualListing___jl0JE")) {
                createUI();
                watchChanges();
                applyFilter(); // initial run
                observer.disconnect();
            }
        });

        observer.observe(document.body, { childList: true, subtree: true });
    }

    waitForPage();
})();
