## Recon 리콘 화살

Recon으로 이름 지어진 활로 화살을 쏘면, 화살이 떨어진 자리를 기준으로 9x9x9범위에 존재하는 플레이어에게 3초 동안 발광 효과를 부여해 위치가 드러나게함

```java
//recon.java
package weapon.weapon.weapons;

import org.bukkit.Bukkit;
import org.bukkit.command.ConsoleCommandSender;
import org.bukkit.entity.Arrow;
import org.bukkit.entity.Entity;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.entity.EntityShootBowEvent;
import org.bukkit.event.entity.ProjectileHitEvent;
import org.bukkit.metadata.FixedMetadataValue;
import org.bukkit.plugin.java.JavaPlugin;
import java.util.Timer;
import java.util.TimerTask;
import java.util.UUID;

public class recon_event implements Listener {
    JavaPlugin plugin;
    ConsoleCommandSender console;
    public recon_event(JavaPlugin plugin,ConsoleCommandSender console){
        this.plugin = plugin;   //메타데이터를 위해 플러그인 객체를 갖고 있어야 함
        this.console = console;
    }
    @EventHandler
    public void recon_events(EntityShootBowEvent e){
        String name = e.getBow().getItemMeta().getDisplayName();
        UUID uuid = e.getEntity().getUniqueId();
        try{
            if(name.equals("recon")){
                Arrow arrow = (Arrow)e.getProjectile();
                arrow.setMetadata("recon", new FixedMetadataValue(plugin, true));
                arrow.setMetadata(uuid.toString(), new FixedMetadataValue(plugin, true)); //누가 쐈는지는 알아야죠?
                arrow.setCritical(true);
                arrow.setDamage(1);
            }
        }catch (Exception error){
            console.sendMessage("Error! Error! Error!");
        }
    }
    @EventHandler
    public void onProjectileHit(ProjectileHitEvent e){
        if(e.getEntity().hasMetadata("recon")){
            Entity arrow = e.getEntity();
            double n = 9; //범위
            double[] e_cor = {arrow.getLocation().getX(),arrow.getLocation().getY(),arrow.getLocation().getZ()}; //화살이 떨어진 위치
            for(Player player : Bukkit.getServer().getOnlinePlayers()){ //전체 플레이어를 검사
                if(!e.getEntity().hasMetadata(player.getUniqueId().toString())){ //화살 쏜 놈은 넘어감
                    double[] cor = {player.getLocation().getX(),player.getLocation().getY(),player.getLocation().getZ()};//그 플레이어의 위치
                    if(range(e_cor[0],cor[0],n)&&range(e_cor[1],cor[1],n)&&range(e_cor[2],cor[2],n)){ //화살 기준으로 9x9x9안에 존재하면 발광
                        player.sendRawMessage("당신은 발각되었다!");
                        player.setGlowing(true);
                        Timer m_timer = new Timer();
                        TimerTask m_task = new TimerTask(){
                            @Override
                            public void run(){
                                player.setGlowing(false);
                            }
                        };
                        m_timer.schedule(m_task, 5000);
                    }else{
                        player.sendRawMessage("범위 안에는 없네용~");
                    }
                }else{
                    player.sendRawMessage("리콘 화살 도착!");
                }
            }
            arrow.remove(); //화살 삭제
        }
    }
    public boolean range(double e_cor,double cor,double x){
        if(e_cor-x <= cor && cor<= e_cor+x){
           return true;
        }else{
            return false;
        }
    }
}
```

일단 command로 만드는 것이 아닌 오직 이벤트 핸들러만을 이용해 구현하였음

두 가지의 이벤트를 이용함, EntityShootBowEvent와 ProjectileHitEvent를 이용하였음

먼저 EntityShootBowEvent는 엔티티가 활 종류를 쏠 때 발생하는 이벤트이고, ProjectileHitEvent는 투사체가 어디에 도착하였을시에 일어남

EntityShootBowEvent를 이용하여 recon활을 쏠시에 화살에다가 recon, 플레이어를 구별하기 위해 쏜 플레이어의 UUID를 메타데이터로 설정하여 보냄

ProjectileHitEvent를 이용해서 화살이 도착했을 때, 모든 플레이어의 좌표값을 화살을 기준으로한 범위에 있는지 확인하고 존재시 3초 동안 발광으로 위치 드러냄