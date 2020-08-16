```java
package me.GRGoose.MicroUHC;

import java.util.Random;


import org.bukkit.Bukkit;
import org.bukkit.ChatColor;
import org.bukkit.Location;
import org.bukkit.World;
import org.bukkit.WorldBorder;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Entity;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.entity.EntityDamageByEntityEvent;
import org.bukkit.plugin.java.JavaPlugin;
import org.bukkit.scheduler.BukkitScheduler;

import net.md_5.bungee.api.ChatMessageType;
import net.md_5.bungee.api.chat.TextComponent;

public class Main extends JavaPlugin implements Listener {
	int phase = 0;
	int willSet = 0;
	boolean a9 = true;
	@Override
	public void onEnable() {
		this.getServer().getPluginManager().registerEvents(this, this);
	}
	@Override
	public void onDisable() {
		
	}
	
	@EventHandler
	public void disablePVP(EntityDamageByEntityEvent pvpevent) {
		if(phase == 0) {
			Entity damaged = pvpevent.getEntity();
			Entity damager = pvpevent.getDamager();
			if(damaged instanceof Player && damager instanceof Player) {
				pvpevent.setCancelled(true);
				damager.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "You cannot PvP during grace period!");
			}
		}
	}
	
	public void broadcastToAll(String msg) {
		for(Player player : Bukkit.getOnlinePlayers()) {
			player.sendMessage(ChatColor.translateAlternateColorCodes('&', msg));
		}
	}
	
	public void actionToAll(String msg) {
		for(Player player : Bukkit.getOnlinePlayers()) {
			player.spigot().sendMessage(ChatMessageType.ACTION_BAR, TextComponent.fromLegacyText(ChatColor.translateAlternateColorCodes('&', msg)));			
		}
	}
	
	public void teleportAllTo(Location loc) {
		for(Player player : Bukkit.getOnlinePlayers()) {
			player.teleport(loc);
		}
	}
	
	public void spreadPlayers(int range1, int range2) {
		int maxX = range1;
		int maxZ = range2;
		Random random = new Random();
		for (Player p : Bukkit.getServer().getOnlinePlayers())
		{
		      int x = random.nextInt(maxX);
		      int z = random.nextInt(maxZ);
		      p.teleport((Location) p.getWorld().getHighestBlockAt(x, z));
		}
	}
	
	public void decreaseBorder(int toSize, int sizeTime, WorldBorder border) {
		double newSize = border.getSize() - toSize;
		border.setSize(newSize, sizeTime);
	}
	
	public String getTimeToString(int sec) {
		int minutesClock = sec / 60;
		int secondsClock = sec % 60;
		String finalClock = "error";
		if(secondsClock < 10) {
			finalClock = Integer.toString(minutesClock) + ":0" + Integer.toString(secondsClock);
		} else if (secondsClock >= 10) {
			finalClock = Integer.toString(minutesClock) + ":" + Integer.toString(secondsClock);
		}
		return finalClock;
	}
	
	public void tickGameTimer(int grace, int timeBetweenDecrease, int sizeToMeetup, int meetupDelay, int sizeEachDecrease, int timeOfDecrease, int meetupBorder, int meetupShrink, boolean positions, WorldBorder border) {
		/* first arg - grace time
		second arg - time between each decrease
		third arg - size of the border before meetup timer is started
		fourth arg - delay between when the border reaches size till the meetup starts
		fifth arg - number of blocks the wb shrinks each decrease
		sixth arg - time the border takes to decrease by <fifth arg> blocks
		seventh arg -  size of border at meetup
		eighth arg - time it takes for the border to shrink to meetup size <seventh arg> */ 
		// phase 0 before grace, lasts 5 mins no pvp - can customize grace time
		// phase 1 when border decreases every 1 min until 100 size - can customize delay between decreases, decrease time and decrease amount
		// phase 2 is delay to meetup
		// meetup is phase 3, border is decreased to 25, get positions - can customize size to start meetup, delay before meetup, border size on meetup, time to shrink to meetup and if can get pos command on meetup
		// if phase is 0, tick timer, disable pvp
		phase = 0;
		int clock[] = {grace};
		@SuppressWarnings("unused")
		int bordersize = 2000;
		BukkitScheduler scheduler = getServer().getScheduler();
	    scheduler.scheduleSyncRepeatingTask(this, new Runnable() {
	        @Override
	        public void run() {
	        	if(phase != 3) {
	        		clock[0]--;
	        	}
	        	if(phase == 0 && clock[0] > 0) {
	        		// phase is grace period and clock is over 0
	        		actionToAll("&b&lGrace period ends in " + getTimeToString(clock[0]));
	        	} else if (phase == 0 && clock[0] < 1) {
	        		// phase is grace periodd and clock reached 0
	        		broadcastToAll("&c&lGrace period is over! You can now PvP, and the border will start decreasing!!");
	        		phase = 1;
	        		clock[0] = timeBetweenDecrease;
	        	} else if (phase == 1 && clock[0] > 0) {
	        		// phase is decreasing and clock is over 0
	        		actionToAll("&6&lBorder decreases to " + willSet + "x" + willSet + " &6&lin " + getTimeToString(clock[0]));
	        	} else if (phase == 1 && clock[0] < 1) {
	        		// phase is decreasing clock is 0
	        		// check if border size will reach meetup size, if so, start meetup
	        		if(border.getSize() == sizeToMeetup + sizeEachDecrease) {
	        			phase = 2;
	        			clock[0] = meetupDelay;
	        			broadcastToAll("&c&lMeetup starts in " + getTimeToString(clock[0]) + "&c&l! During meetup the border will shrink to " + Integer.toString(meetupBorder) + "&c&l and you will have to fight!");
	        		}
	        		// decrease border
	        		decreaseBorder(sizeEachDecrease, timeOfDecrease, border);
	        		willSet = willSet - sizeEachDecrease;
	        		clock[0] = timeBetweenDecrease;
	        	} else if (phase == 2 && clock[0] > 0) {
	        		// phase is delay to meetup and clock is over 0
	        		actionToAll("&c&lMeetup starts in " + getTimeToString(clock[0]));
	        	} else if (phase == 2 && clock[0] < 1) {
	        		// meetup start
	        		phase = 3;
	        		clock[0] = 0;
	        		border.setSize(meetupBorder, meetupShrink);
	        		broadcastToAll("&4&lThe border is shrinking to " + Integer.toString(meetupBorder) + " and meetup has started! During meetup you will have to fight to the death!");
	        		if(positions == true) {
	        			broadcastToAll("&4&lDuring meetup you can execute /getstatus <player> to get the position, health and hunger of a specific player! Use the command to find and kill other players!");
	        		}
	        	} else if (phase == 3) {
	        		//meetup timer
	        		clock[0]++;
	        		actionToAll("&4&lTime elapsed during meetup: " + getTimeToString(clock[0]));
	        	}
	        }
	    }, 0L, 20L);
	}
	
	public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args) {
		if(label.equalsIgnoreCase("microuhc")) {
			Player admin = (Player) sender;
			World uhcworld = admin.getWorld();
			WorldBorder border = uhcworld.getWorldBorder();
			Location startLoc = admin.getLocation();
			border.setCenter(startLoc);
			willSet = Integer.parseInt(args[0]);
			@SuppressWarnings("unused")
			int spreadRange = willSet - 200;
			border.setSize(willSet);
			teleportAllTo(startLoc);
			int a1 = Integer.parseInt(args[1]);
			int a2 = Integer.parseInt(args[2]);
			int a3 = Integer.parseInt(args[3]);
			int a4 = Integer.parseInt(args[4]);
			int a5 = Integer.parseInt(args[5]);
			int a6 = Integer.parseInt(args[6]);
			int a7 = Integer.parseInt(args[7]);
			int a8 = Integer.parseInt(args[8]);
			a9 = Boolean.valueOf(args[9]);
			willSet = willSet - 100;
			tickGameTimer(a1,a2,a3,a4,a5,a6,a7,a8,a9,border);
			return true;
		}
		if(label.equalsIgnoreCase("getstatus")) {
			if(a9 == true) {
				if(phase != 3) {
					sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "You cannot use this command yet! Please wait until meetup!");
					return true;
				}
				Player getted = (Player) Bukkit.getServer().getPlayer(args[0]);
				if(getted == null) {
					sender.sendMessage(ChatColor.RED + "That player is not online!");
					return true;
				}
				sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "Displaying status of player " + args[0] + ":");
				sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "Health:" + ChatColor.RESET + "" + ChatColor.RED + " " + getted.getHealth());
				sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "Hunger:" + ChatColor.RESET + "" + ChatColor.RED + " " + getted.getFoodLevel());
				Location statusLoc = getted.getLocation();
				int statusX = statusLoc.getBlockX();
				int statusY = statusLoc.getBlockY();
				int statusZ = statusLoc.getBlockZ();
				sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "Position: " + ChatColor.RESET + "" + ChatColor.RED + "X " + Integer.toString(statusX) + " Y " + Integer.toString(statusY) + " Z " + Integer.toString(statusZ));
				return true;
			} else {
				sender.sendMessage(ChatColor.RED + "" + ChatColor.BOLD + "This command was disabled for this UHC!");
			}
		}
		return false;
	}
}```
