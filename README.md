package パッケージ

import org.bukkit.Bukkit;
import org.bukkit.ChatColor;
import org.bukkit.Material;
import org.bukkit.Sound;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.inventory.InventoryClickEvent;
import org.bukkit.inventory.Inventory;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.meta.ItemMeta;
import org.bukkit.inventory.meta.SkullMeta;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

public class Adminguiplugin extends JavaPlugin implements Listener {

    private final Map<UUID, String> messageInputtingPlayers = new HashMap<>();

    // プラグインの有効化時の処理
    @Override
    public void onEnable() {
        this.getLogger().info("AdminGUIプラグインが有効になりました！");
        getServer().getPluginManager().registerEvents(this, this);
    }

    // プラグインの無効化時の処理
    @Override
    public void onDisable() {
        this.getLogger().info("AdminGUIプラグインが無効になりました。");
    }

    // コマンドの処理
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {
        if (sender instanceof Player) {
            Player player = (Player) sender;
            if (command.getName().equalsIgnoreCase("admingui")) {
                openAdminGUI(player);
                return true;
            }
        }
        return false;
    }

    // GUIを開くメソッド
    private void openAdminGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 36, "Admin GUI"); // インベントリサイズを36に変更

        // アイテム1: サーバー再起動ボタン
        ItemStack restartButton = createGuiItem(Material.REDSTONE_BLOCK, "§cサーバー再起動", "クリックでサーバーを再起動します");
        gui.setItem(10, restartButton);

        // アイテム2: プレイヤーをキックするボタン
        ItemStack kickButton = createGuiItem(Material.BARRIER, "§eプレイヤーをキック", "クリックでキックするプレイヤーを選択します");
        gui.setItem(12, kickButton);

        // アイテム3: 全プレイヤーにメッセージを送るボタン
        ItemStack messageButton = createGuiItem(Material.PAPER, "§a全体にメッセージ送信", "クリックでメッセージを入力します");
        gui.setItem(14, messageButton);

        // アイテム4: 時間設定ボタン
        ItemStack timeButton = createGuiItem(Material.CLOCK, "§b時間設定", "クリックで時間を設定します");
        gui.setItem(16, timeButton);

        // アイテム5: 天候変更ボタン
        ItemStack weatherButton = createGuiItem(Material.WATER_BUCKET, "§d天候変更", "クリックで天候を設定します");
        gui.setItem(28, weatherButton);

        // アイテム6: ゲームモード変更ボタン
        ItemStack gamemodeButton = createGuiItem(Material.IRON_SWORD, "§3ゲームモード変更", "クリックでゲームモードを設定します");
        gui.setItem(30, gamemodeButton);

        // アイテム7: 経験値付与ボタン
        ItemStack expButton = createGuiItem(Material.EXPERIENCE_BOTTLE, "§a経験値付与", "クリックで経験値を付与します");
        gui.setItem(32, expButton);

        // アイテム8: プレイヤーへのテレポートボタン
        ItemStack teleportButton = createGuiItem(Material.ENDER_PEARL, "§5テレポート", "クリックでテレポートするプレイヤーを選択します");
        gui.setItem(34, teleportButton);



        // GUIを開く
        player.openInventory(gui);
    }

    // GUIアイテムを作成するヘルパーメソッド
    private ItemStack createGuiItem(Material material, String displayName, String... lore) {
        ItemStack item = new ItemStack(material);
        ItemMeta meta = item.getItemMeta();
        if (meta != null) {
            meta.setDisplayName(displayName);
            meta.setLore(java.util.Arrays.asList(lore));
            item.setItemMeta(meta);
        }
        return item;
    }

    // インベントリのクリックイベントの処理
    @EventHandler
    public void onInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("Admin GUI")) return;

        event.setCancelled(true); // アイテムの移動を防ぐ

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || !clickedItem.hasItemMeta()) return;

        String displayName = clickedItem.getItemMeta().getDisplayName();

        // サーバー再起動ボタンがクリックされた場合の処理
        if (displayName.equals("§cサーバー再起動")) {
            player.sendMessage("§cサーバーを再起動します...");
            Bukkit.getServer().shutdown();
        }
        // プレイヤーをキックするボタンがクリックされた場合の処理
        else if (displayName.equals("§eプレイヤーをキック")) {
            openKickPlayerGUI(player);
        }
        // 全プレイヤーにメッセージを送るボタンがクリックされた場合の処理
        else if (displayName.equals("§a全体にメッセージ送信")) {
            messageInputtingPlayers.put(player.getUniqueId(), "message_broadcast");
            player.sendMessage(ChatColor.GREEN + "チャットでメッセージを入力してください。");
            player.closeInventory();
        }
        // 時間設定ボタンがクリックされた場合の処理
        else if (displayName.equals("§b時間設定")) {
            openSetTimeGUI(player);
        }
        // 天候変更ボタンがクリックされた場合の処理
        else if (displayName.equals("§d天候変更")) {
            openSetWeatherGUI(player);
        }
        // ゲームモード変更ボタンがクリックされた場合の処理
        else if(displayName.equals("§3ゲームモード変更")){
            openSetGamemodeGUI(player);
        }
        // 経験値付与ボタンがクリックされた場合の処理
        else if(displayName.equals("§a経験値付与")){
            openSetExpGUI(player);
        }
        // テレポートボタンがクリックされた場合の処理
        else if(displayName.equals("§5テレポート")){
            openTeleportPlayerGUI(player);
        }
    }
    // チャットイベントの処理
    @EventHandler
    public void onPlayerChat(org.bukkit.event.player.AsyncPlayerChatEvent event){
        Player player = event.getPlayer();
        UUID uuid = player.getUniqueId();
        String message = event.getMessage();

        if(messageInputtingPlayers.containsKey(uuid)){
            String mode = messageInputtingPlayers.get(uuid);
            if(mode.equals("message_broadcast")){
                event.setCancelled(true);
                Bukkit.broadcastMessage(ChatColor.GREEN + "[全体メッセージ]" + ChatColor.RESET + message);
                messageInputtingPlayers.remove(uuid);
            }else if(mode.equals("exp")){
                event.setCancelled(true);
                try{
                    int expAmount = Integer.parseInt(message);
                    player.giveExp(expAmount);
                    player.sendMessage(ChatColor.GREEN + String.valueOf(expAmount) +"の経験値を付与しました。");
                }catch (NumberFormatException e){
                    player.sendMessage(ChatColor.RED + "無効な数字です。");
                }
                messageInputtingPlayers.remove(uuid);
            }
        }
    }


    private void openSetTimeGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 9, "時間設定");

        // 昼にするボタン
        ItemStack dayButton = createGuiItem(Material.SUNFLOWER, "§6昼", "クリックで時間を昼にします");
        gui.setItem(0, dayButton);

        // 夜にするボタン
        ItemStack nightButton = createGuiItem(Material.SNOWBALL, "§9夜", "クリックで時間を夜にします");
        gui.setItem(2, nightButton);

        player.openInventory(gui);
    }

    private void openSetWeatherGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 9, "天候設定");

        // 晴れにするボタン
        ItemStack sunButton = createGuiItem(Material.SUNFLOWER, "§e晴れ", "クリックで天候を晴れにします");
        gui.setItem(0, sunButton);

        // 雨にするボタン
        ItemStack rainButton = createGuiItem(Material.WATER_BUCKET, "§b雨", "クリックで天候を雨にします");
        gui.setItem(2, rainButton);

        // 雷雨にするボタン
        ItemStack thunderButton = createGuiItem(Material.IRON_BARS, "§d雷雨", "クリックで天候を雷雨にします");
        gui.setItem(4, thunderButton);

        player.openInventory(gui);
    }

    private void openSetGamemodeGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 9, "ゲームモード設定");

        // サバイバルにするボタン
        ItemStack survivalButton = createGuiItem(Material.IRON_PICKAXE, "§aサバイバル", "クリックでゲームモードをサバイバルにします");
        gui.setItem(0, survivalButton);

        // クリエイティブにするボタン
        ItemStack creativeButton = createGuiItem(Material.GRASS_BLOCK, "§bクリエイティブ", "クリックでゲームモードをクリエイティブにします");
        gui.setItem(2, creativeButton);

        // アドベンチャーにするボタン
        ItemStack adventureButton = createGuiItem(Material.MAP, "§dアドベンチャー", "クリックでゲームモードをアドベンチャーにします");
        gui.setItem(4, adventureButton);

        // スペクテイターにするボタン
        ItemStack spectatorButton = createGuiItem(Material.ENDER_EYE, "§eスペクテイター", "クリックでゲームモードをスペクテイターにします");
        gui.setItem(6, spectatorButton);

        player.openInventory(gui);
    }

    private void openSetExpGUI(Player player){
        messageInputtingPlayers.put(player.getUniqueId(), "exp");
        player.sendMessage(ChatColor.GREEN + "チャットで経験値を入力してください。");
        player.closeInventory();
    }

    private void openKickPlayerGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 54, "キックするプレイヤーを選択");

        int slot = 0;
        for (Player onlinePlayer : Bukkit.getOnlinePlayers()) {
            ItemStack playerHead = new ItemStack(Material.PLAYER_HEAD);
            ItemMeta meta = playerHead.getItemMeta();
            if (meta != null) {
                meta.setDisplayName(onlinePlayer.getName());
                java.util.List<String> lore = java.util.Arrays.asList("クリックでキック");
                meta.setLore(lore);
                playerHead.setItemMeta(meta);
                SkullMeta skullMeta = (SkullMeta) meta;
                skullMeta.setOwningPlayer(onlinePlayer);
                playerHead.setItemMeta(skullMeta);

                gui.setItem(slot, playerHead);
                slot++;
            }
        }
        player.openInventory(gui);
    }

    private void openTeleportPlayerGUI(Player player) {
        Inventory gui = Bukkit.createInventory(null, 54, "テレポートするプレイヤーを選択");

        int slot = 0;
        for (Player onlinePlayer : Bukkit.getOnlinePlayers()) {
            if(onlinePlayer.equals(player)) continue;
            ItemStack playerHead = new ItemStack(Material.PLAYER_HEAD);
            ItemMeta meta = playerHead.getItemMeta();
            if (meta != null) {
                meta.setDisplayName(onlinePlayer.getName());
                java.util.List<String> lore = java.util.Arrays.asList("クリックでテレポート");
                meta.setLore(lore);
                playerHead.setItemMeta(meta);
                SkullMeta skullMeta = (SkullMeta) meta;
                skullMeta.setOwningPlayer(onlinePlayer);
                playerHead.setItemMeta(skullMeta);

                gui.setItem(slot, playerHead);
                slot++;
            }
        }
        player.openInventory(gui);
    }


    // キックするプレイヤーのGUIのクリックイベント処理
    @EventHandler
    public void onKickPlayerInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("キックするプレイヤーを選択")) return;

        event.setCancelled(true);

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || clickedItem.getType() != Material.PLAYER_HEAD || !clickedItem.hasItemMeta()) {
            return;
        }

        ItemMeta meta = clickedItem.getItemMeta();
        String playerName = meta.getDisplayName();

        Player target = Bukkit.getPlayer(playerName);

        if (target != null) {
            target.kickPlayer("§c管理者によってキックされました。");
            player.sendMessage(ChatColor.RED + target.getName() + "をキックしました。");
            player.playSound(player.getLocation(), Sound.ENTITY_EXPERIENCE_ORB_PICKUP, 1, 1);
        } else {
            player.sendMessage(ChatColor.RED + "プレイヤーが見つかりませんでした。");
        }
        player.closeInventory();
    }
    //テレポートするプレイヤーのGUIクリックイベント
    @EventHandler
    public void onTeleportPlayerInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("テレポートするプレイヤーを選択")) return;

        event.setCancelled(true);

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || clickedItem.getType() != Material.PLAYER_HEAD || !clickedItem.hasItemMeta()) {
            return;
        }

        ItemMeta meta = clickedItem.getItemMeta();
        String playerName = meta.getDisplayName();

        Player target = Bukkit.getPlayer(playerName);

        if (target != null) {
            player.teleport(target);
            player.sendMessage(ChatColor.GREEN + target.getName() + "にテレポートしました。");
            player.playSound(player.getLocation(), Sound.ENTITY_ENDERMAN_TELEPORT, 1, 1);
        } else {
            player.sendMessage(ChatColor.RED + "プレイヤーが見つかりませんでした。");
        }
        player.closeInventory();
    }


    @EventHandler
    public void onSetTimeInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("時間設定")) return;

        event.setCancelled(true);

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || !clickedItem.hasItemMeta()) {
            return;
        }

        String displayName = clickedItem.getItemMeta().getDisplayName();

        if (displayName.equals("§6昼")) {
            player.getWorld().setTime(6000);
            player.sendMessage("§6時間を昼に設定しました。");
            player.playSound(player.getLocation(), Sound.BLOCK_NOTE_BLOCK_PLING, 1, 1);
        } else if (displayName.equals("§9夜")) {
            player.getWorld().setTime(18000);
            player.sendMessage("§9時間を夜に設定しました。");
            player.playSound(player.getLocation(), Sound.BLOCK_NOTE_BLOCK_PLING, 1, 1);
        }

        player.closeInventory();
    }
    @EventHandler
    public void onSetWeatherInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("天候設定")) return;

        event.setCancelled(true);

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || !clickedItem.hasItemMeta()) {
            return;
        }

        String displayName = clickedItem.getItemMeta().getDisplayName();

        if (displayName.equals("§e晴れ")) {
            player.getWorld().setStorm(false);
            player.getWorld().setThundering(false);
            player.sendMessage("§e天候を晴れに設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        } else if (displayName.equals("§b雨")) {
            player.getWorld().setStorm(true);
            player.getWorld().setThundering(false);
            player.sendMessage("§b天候を雨に設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);

        } else if(displayName.equals("§d雷雨")){
            player.getWorld().setStorm(true);
            player.getWorld().setThundering(true);
            player.sendMessage("§d天候を雷雨に設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        }

        player.closeInventory();
    }
    @EventHandler
    public void onSetGamemodeInventoryClick(InventoryClickEvent event) {
        if (!event.getView().getTitle().equals("ゲームモード設定")) return;

        event.setCancelled(true);

        Player player = (Player) event.getWhoClicked();
        ItemStack clickedItem = event.getCurrentItem();

        if (clickedItem == null || !clickedItem.hasItemMeta()) {
            return;
        }

        String displayName = clickedItem.getItemMeta().getDisplayName();
        if (displayName.equals("§aサバイバル")) {
            player.setGameMode(org.bukkit.GameMode.SURVIVAL);
            player.sendMessage("§aゲームモードをサバイバルに設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        } else if (displayName.equals("§bクリエイティブ")) {
            player.setGameMode(org.bukkit.GameMode.CREATIVE);
            player.sendMessage("§bゲームモードをクリエイティブに設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        } else if(displayName.equals("§dアドベンチャー")){
            player.setGameMode(org.bukkit.GameMode.ADVENTURE);
            player.sendMessage("§dゲームモードをアドベンチャーに設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        } else if(displayName.equals("§eスペクテイター")){
            player.setGameMode(org.bukkit.GameMode.SPECTATOR);
            player.sendMessage("§eゲームモードをスペクテイターに設定しました。");
            player.playSound(player.getLocation(), Sound.ENTITY_VILLAGER_YES, 1, 1);
        }

        player.closeInventory();
    }
}
