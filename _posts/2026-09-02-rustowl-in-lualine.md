---
layout: post
title:  "Rustowl in Lualine"
date:   2026-09-02 09:59:00 +0700
categories: rust neovim rustowl lualine
---
If you don't know what is Rustowl, go head to the project repo [https://github.com/cordx56/rustowl](https://github.com/cordx56/rustowl).

We gonna show Rustowl status on Lualine (see: [https://github.com/nvim-lualine/lualine.nvim](https://github.com/nvim-lualine/lualine.nvim)).
The reason why we want to have it is because we might not aware whether Rustowl is enabled or not, so that showing the Rustowl on Lualine will give an extra feedback to show the Rustowl status. 

Make sure you have Rustowl properly setup on your Neovim.


The plugin setup:
```lua
-- filename: lualine.lua (loaded by lazy.nvim)
return {
    "nvim-lualine/lualine.nvim",
    dependencies = {
        "nvim-tree/nvim-web-devicons", -- or you can use mini.icons from mini.nvim
    },
    event = "VeryLazy",
    opts = {
    sections = {
        lualine_x = {
            {
                function()
                    local ok, rustowl = pcall(require, "rustowl")
                    if not ok then
                        return ""
                    end

                    return rustowl.is_enabled() and "Rustowl" or ""
                end,
                cond = function()
                    return vim.bo.filetype == "rust"
                end,
            },

            -- built-in components, not related to rustowl status
            "encoding",
            "fileformat",
            "filetype",
        },
    }
    },
}
```

Above is minimum setup that I used.

Try to enable/disable the rustowl through Neovim command line `:Rustowl toggle`, it will show the the Rustowl status on Lualine.

Alternatively, if you want to have colors and mouse interaction:
```lua
-- filename: lualine.lua (loaded by lazy.nvim)
return {
    "nvim-lualine/lualine.nvim",
    dependencies = {
        "nvim-tree/nvim-web-devicons", -- or you can use mini.icons from mini.nvim
    },
    event = "VeryLazy",
    opts = {
    sections = {
        lualine_x = {
            -- Rustowl status component
            {
                function()
                    local ok, rustowl = pcall(require, "rustowl")
                    if not ok then
                        return ""
                    end

                    return rustowl.is_enabled() and "Rustowl on" or "Rustowl off"
                end,
                cond = function()
                    return vim.bo.filetype == "rust"
                end,
                color = function()
                    local ok, palette = pcall(function()
                        return require("catppuccin.palettes").get_palette()
                    end)
                    local green = ok and palette.green or "#a6e3a1" 
                    local dim = ok and palette.overlay0 or "#6c7086"

                    local ok_owl, rustowl = pcall(require, "rustowl")
                    if ok_owl and rustowl.is_enabled() then
                        return { fg = green }
                    end
                    return { fg = dim }
                end,
                on_click = function()
                    local ok, rustowl = pcall(require, "rustowl")
                    if ok then
                        rustowl.toggle()
                        require("lualine").refresh()
                    end
                end,
            },

            -- below is not related to rustowl status
            "encoding",
            "fileformat",
            "filetype",
        },
    }
    },
}
```

Here now you can see the Rustowl status is colorized. Also you can interact with mouse.
