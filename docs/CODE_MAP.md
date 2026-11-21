# Bản đồ source server game (Nro-Online / Bladekun)

Cập nhật: 2026-11-21 · repo https://github.com/gnoudeath/Nro-Online · commit `6a03fab` (có thể đã cũ, kiểm tra `git log`).
Quy mô: 441 file `.java`, khoảng 97.000 dòng, Java 17 + Maven, class chạy: `nro.server.ServerManager`.
Mục đích tài liệu: tra nhanh "code X nằm ở file nào / luồng Y đi qua đâu" mà không phải đọc lại cả dự án.
Đây là chỉ mục, không thay cho đọc code: khi cần chi tiết, mở đúng file được chỉ tới.

## Thư mục ngoài src
- `config/server.properties` (db, cổng 14445, server.sv1, domain, event) · `config/game.properties` · `config/event-he.txt`
- `ServerLogin/` server đăng nhập riêng (chỉ có jar + server.ini + pom, KHÔNG có mã nguồn). Cổng 8888, đọc bảng `account`.
- `Sql/Sql Nro.sql` database (tên DB dùng là `nro-d`) · `resources/`, `data/`, `Eff/` dữ liệu và ảnh cho client · `log/` (chỉ .gitkeep)

## Luồng chính (đã kiểm tra trong code)
1. **Khởi động** `ServerManager.main` → `ServerManager.gI()` → `init()`: `Manager.gI()` (đọc `config/server.properties` qua `Manager.loadProperties`, nạp dữ liệu game từ DB), xóa lịch sử giao dịch, `BossFactory.initBoss()`, tạo `Controller`.
   Sau đó `run()`: mở cửa sổ Swing (`panel`), rồi `activeCommandLine()` (lệnh console: `baotri`, `athread`, `nplayer`, `a`), `activeGame()` (thread cập nhật boss, doanh trại, bản đồ kho báu, mỗi 500ms), `activeLogin()` (nối tới server login), `autoTask()`, `AutoMaintenance(23,58,59)` (bảo trì tự động mỗi ngày), `activeServerSocket()` (nhận kết nối client ở `PORT`, chặn theo IP bằng `Manager.MAX_PER_IP`).
2. **Kết nối client**: mỗi socket tạo một `server.io.Session`, có 2 thread: `MessageSender` (gửi) và `MessageCollector` (nhận). Nhận được `Message` thì gọi `Controller.onMessage(session, msg)`.
3. **Điều phối lệnh**: `server.Controller.onMessage` là switch lớn theo `msg.command` (hằng ở `nro.consts.Cmd`, nhiều số cứng như -127, 112). Mỗi case gọi vào `services.*` tương ứng. Có `messageNotLogin` (chưa đăng nhập: case 0 = đăng nhập `session.login(user, pass)`), `messageNotMap` (chưa vào map), `messageSubCommand`.
4. **Đăng nhập**: Controller → `Session.login` → gửi qua `login.LoginSession` (socket tới server login, `Manager.loginHost`/`loginPort`, mặc định 127.0.0.1:8888) → server login kiểm tra bảng `account` → trả kết quả về → `Controller.sendInfo`/tạo nhân vật (`createChar`).
5. **Database**: `jdbc.DBService` (cấu hình DB_HOST/PORT/NAME/USER/PASSWORD, ghi đè từ `server.db.*`), `jdbc.DBHika` (HikariCP, cứng 20 idle/100 max), `jdbc.ConnPool`; truy vấn dữ liệu người chơi nằm ở `jdbc.daos.*` (PlayerDAO, AccountDAO…).
6. **Top**: `services.TopService` (Top SD chỉ online, Top Nhiệm vụ đọc `data_task`, Top VND đọc `account.tongnap`).
7. **Dữ liệu người chơi** lưu trong bảng `player` dạng JSON: `data_point` (chỉ số gốc: [0]=giới hạn sức mạnh, [1]=sức mạnh, [2]=tiềm năng, [3]=thể lực, [4]=thể lực đầy, [5]=HP gốc, [6]=Ki gốc, [7]=sức đánh gốc, [8]=giáp, [9]=chí mạng…), `data_task` = [tiến độ, nhiệm vụ chính, bước nhỏ], `items_body` (trang bị đang mặc), `pet_*` (đệ tử).
8. **Gửi cho client danh sách server**: `data.DataGame.LINK_IP_PORT` (tạo từ `server.sv1`… trong `Manager`).

## Điểm cần nhớ
- `Controller.CLIENT_SHARED_SECRET` là khóa bắt tay với client: không đổi nếu không đổi client.
- Cổng API HTTP (`nro.netty`) đã bị chú thích, không chạy.
- `BotManager.createBot` bị chú thích (không có bot).
- Đã dọn: bỏ domain/IP/API lạ, tên server Bladekun, DB `nro-d` (commit `6a03fab`).
- Web liên quan: repo `gnoudeath/bladekun-web` (PHP MVC), dùng chung DB.

# Chỉ mục file (định dạng: file (số dòng) [extends]: tên hàm)

## nro.attr
- Attribute.java (77): Attribute, update, setChanged, isExpired, toString, setValue, setTime, setTemplate
- AttributeManager.java (76): AttributeManager, add, remove, find, update, setTime
- AttributeTemplate.java (24): 
- AttributeTemplateManager.java (67): getInstance, load, add, remove, find

## nro.card
- Card.java (46): addAmount, levelUp, setTemplate
- CardManager.java (110): getInstance, load, add, remove, find
- CardTemplate.java (41): 
- CollectionBook.java (80): CollectionBook, init, add, remove, find, findWithItemID, findWithMobID

## nro.consts
- BuyLimit.java (21): 
- Cmd.java (305): 
- ConstAccount.java (18): 
- ConstAchive.java (24): 
- ConstAdminCommand.java (25): 
- ConstAttribute.java (20): 
- ConstEvent.java (15): 
- ConstIgnoreName.java (20): 
- ConstItem.java (1327): 
- ConstMap.java (170): 
- ConstMob.java (125): 
- ConstNpc.java (456): 
- ConstPlayer.java (32): 
- ConstRatio.java (21): 
- ConstRewardLimit.java (15): 
- ConstTask.java (425): 
- ConstTextColor.java (15): 
- ConstTranhNgocNamek.java (25): 
- MapName.java (171): 

## nro.core.concurrent
- GameScheduler.java (52): GameScheduler, shutdown

## nro.core.loop
- GameLoop.java (88): Entry, GameLoop, gI, start, stop, register, unregister, run
- Tickable.java (45): isActive

## nro.data
- DataGame.java (303): sendVersionGame, updateData, createMap, updateSkill, sendDataImageVersion, sendDataItemBG, sendTileSetInfo, sendMapTemp, sendHeadAvatar, sendLinkIP
- ItemData.java (146): updateItem, updateTocBay, updateItemOptionItemplate, updateItemTemplate

## nro.dialog
- ConfirmDialog.java (61): ConfirmDialog, show, run, cancel
- MenuDialog.java (55) extends ConfirmDialog: MenuDialog, show, run
- MenuRunable.java (14): 

## nro.event
- Event.java (45): getInstance, initEvent, isEvent
- SummerEvent.java (519) extends Event: init, generateUniqueID, initNpc, openBaseMenu, confirmMenu, initMap, dropItem, useItem, useBonTam, insectTrapping

## nro.jdbc
- ConnPool.java (96): gI, ConnPool, newConnection, getConnection, close, free
- DBHika.java (58): getConnection, close
- DBService.java (211): gI, DBService, getConnectionForLogin, getConnectionForLogout, getConnectionForSaveData, getConnectionForGame, getConnectionForClan, getConnectionForAutoSave, getConnectionForSaveHistory, getConnectionForGetPlayer, getConnectionCreatPlayer, getConnection, release, currentActive, currentIdle

## nro.jdbc.daos
- AccountDAO.java (125): updateAccount, updateLastTimeLoginAllAccount, updateAccoutLogout, banAccount, createAccount
- GodGK.java (2069): login, loadPlayer, loadPlayerbyId
- HistoryTransactionDAO.java (135): insert, deleteHistory
- PlayerDAO.java (1946): createNewPlayer, updatePlayer, saveName, isExistName, checkVnd, subVnd, CongVnd, CongKillBoss, congDiemEven2thang9, CongDiemSuKien, CongTongNap, NangCapHoiVienVIP, ChuyenCan, CheckQuaChuyenCan, CongPointDiemDanh, NapLanDau, PhatThuongOfline, XoaSoMayMan, subRuby, subGoldBar, subActive, addHistoryReceiveGoldBar …+2
- ShopDAO.java (144): getShops, loadShopTab, loadItemShop, loadItemShopOption

## nro.lib
- RandomCollection.java (44): RandomCollection, add, next

## nro.login
- Cmd.java (21): 
- LoginController.java (189): LoginController, process, updateTimeLogout, serverMessage, disconnect, login, onConnectionFail, onConnectOK, onDisconnected
- LoginService.java (88): LoginService, login, logout, setServer, sendMessage
- LoginSession.java (348): LoginSession, connect, reconnect, NetworkInit, run, Thread, doConnect, sendMessage, doSendMessage, readKey, writeKey, Sender, AddMessage, getKey, readMessage, close, cleanNetwork

## nro.manager
- AbsManager.java (31): add, remove, getList
- AchiveManager.java (65): getInstance, load, findByID, add, remove
- ConsignManager.java (102): getInstance, load, close
- IManager.java (15): 
- MiniPetManager.java (64): gI, load, add, remove, findByID
- NamekBallManager.java (79) extends AbsManager: gI, initBall, initFossil, findByID, findByIndex, getIndex
- PetFollowManager.java (51) extends AbsManager: gI, load, findByID
- TopManager.java (494): getInstance, load, load1, load2, load3, loadSieuHang

## nro.models
- Part.java (28): getIcon
- PartImage.java (30): PartImage
- PartManager.java (46): PartPot, PartDetail

## nro.models.boss
- Boss.java (1079) extends Player: changeStatus, Boss, init, version, initSkill, BaoHpBoss, update, getListPlayerAttack, nextMabu, injured, notifyPlayeKill, injuredNotCheckDie, getSkillAttack, getSkillSpecial, getSkillById, die, joinMap, getMapCanJoin, getMapJoin, trieuHoiBoss, joinZone, leaveMap …+34
- BossData.java (2893): BossData
- BossFactory.java (739): BossFactory, initBoss, initBossMabuWar14H, initBossMabuWar, createBoss
- BossManager.java (351): updateAllBoss, BossManager, gI, getBossById, getNewBossInstanceId, getBossByInstanceId, isMultiInstanceBoss, getBossByIdRandom, getBossidlist, addBoss, getBosses, removeBoss, showListBoss, showListBossMember, isVipBoss, showListBossVIP
- FutureBoss.java (17) extends Boss: FutureBoss, injured

## nro.models.boss.BossMoi
- DaiThanh.java (105) extends Boss: DaiThanh, useSpecialSkill, attack, idle, checkPlayerDie, initTalk, leaveMap, rewards
- GiamNguc.java (103) extends Boss: GiamNguc, useSpecialSkill, attack, idle, checkPlayerDie, initTalk, leaveMap, rewards
- Gogeta.java (103) extends Boss: Gogeta, useSpecialSkill, attack, idle, checkPlayerDie, initTalk, leaveMap, rewards
- RobotHuyDiet.java (103) extends Boss: RobotHuyDiet, useSpecialSkill, attack, idle, checkPlayerDie, initTalk, leaveMap, rewards
- Soi_3_Dau.java (103) extends Boss: Soi_3_Dau, useSpecialSkill, attack, idle, checkPlayerDie, initTalk, rewards, leaveMap

## nro.models.boss.BossMoiLam
- Bido.java (101) extends Boss: Bido, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Bill_Bi_Ngo.java (106) extends Boss: Bill_Bi_Ngo, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Brojack.java (105) extends Boss: Brojack, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Bujin.java (103) extends Boss: Bujin, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Coller_Gold.java (102) extends Boss: Coller_Gold, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Hatchiyack.java (107) extends Boss: Hatchiyack, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Heart_Gold.java (100) extends Boss: Heart_Gold, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Kogu.java (104) extends Boss: Kogu, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SuperBrojack.java (103) extends Boss: SuperBrojack, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SuperZamasu.java (107) extends Boss: SuperZamasu, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Thong_Che_Kilo.java (106) extends Boss: Thong_Che_Kilo, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Zangya.java (103) extends Boss: Zangya, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.BossNew
- Along.java (93) extends Boss: Along, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Kaido.java (94) extends Boss: Kaido, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- LufyThanNika.java (94) extends Boss: LufyThanNika, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Mihalk.java (95) extends Boss: Mihalk, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- TienHacAm.java (94) extends Boss: TienHacAm, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.KhungLong
- KhungLong.java (94) extends Boss: KhungLong, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong1.java (90) extends Boss: KhungLong1, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong2.java (90) extends Boss: KhungLong2, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong3.java (90) extends Boss: KhungLong3, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong4.java (90) extends Boss: KhungLong4, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong5.java (89) extends Boss: KhungLong5, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong6.java (89) extends Boss: KhungLong6, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- KhungLong7.java (89) extends Boss: KhungLong7, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.NguHanhSon
- BatGioi.java (82) extends Boss: BatGioi, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- NgoKhong.java (84) extends Boss: NgoKhong, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.NgucTu
- Cumber.java (56) extends FutureBoss: Cumber, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- GokuSuper.java (90) extends Boss: GokuSuper, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- SuperCumber.java (49) extends FutureBoss: SuperCumber, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.bill
- Bill.java (71) extends Boss: Bill, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Whis.java (83) extends Boss: Whis, useSpecialSkill, rewards, idle, leaveMap, checkPlayerDie, initTalk

## nro.models.boss.boss_ban_do_kho_bau
- BossBanDoKhoBau.java (106) extends Boss: BossBanDoKhoBau, spawn, attack, idle, checkPlayerDie, initTalk, leaveMap, rewards, useSpecialSkill, notifyPlayeKill, joinMap
- TrungUyXanhLo.java (101) extends BossBanDoKhoBau: TrungUyXanhLo, attack, useSpecialSkill, rewards, joinMap

## nro.models.boss.boss_doanh_trai
- BossDoanhTrai.java (106) extends Boss: BossDoanhTrai, spawn, idle, checkPlayerDie, initTalk, leaveMap, rewards, useSpecialSkill, notifyPlayeKill
- NinjaAoTim.java (130) extends BossDoanhTrai: NinjaAoTim, attack, idle, rewards, joinMap, phanThan, initTalk
- NinjaAoTimFake.java (63) extends NinjaAoTim: NinjaAoTimFake, attack, rewards
- RobotVeSi.java (69) extends BossDoanhTrai: RobotVeSi, attack, rewards, joinMap
- TrungUyThep.java (90) extends BossDoanhTrai: TrungUyThep, attack, rewards, joinMap, injured
- TrungUyTrang.java (113) extends BossDoanhTrai: TrungUyTrang, getPlayerAttack, attack, injured, rewards, joinMap, useSpecialSkill
- TrungUyXanhLo.java (97) extends BossDoanhTrai: TrungUyXanhLo, attack, useSpecialSkill, rewards, joinMap

## nro.models.boss.bosstuonglai
- BLACKROSE.java (73) extends Boss: BLACKROSE, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Blackgoku.java (89) extends Boss: Blackgoku, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- BongBang.java (97) extends Boss: BongBang, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Itachi.java (97) extends Boss: Itachi, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- MabuTuongLai.java (95) extends Boss: MabuTuongLai, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- RongDen.java (97) extends Boss: RongDen, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- SoiBasil.java (99) extends Boss: SoiBasil, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Superblackgoku.java (89) extends Boss: Superblackgoku, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- WhisDetu.java (95) extends Boss: WhisDetu, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- ZamasToiThuong.java (90) extends Boss: ZamasToiThuong, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- ZenoDetu.java (95) extends Boss: ZenoDetu, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.broly
- Broly.java (381) extends Boss: Broly, initTalk, attack, idle, injured, getPlayerAttack, addPlayerAttack, charge, goToXY, effectCharger, angry, isInListPlayersAttack, checkPlayerDie, joinMap, respawn, getMapCanJoin, leaveMap, die, rewards, useSpecialSkill
- SuperBroly.java (187) extends Broly: SuperBroly, injured, attack, getPlayerAttack, leaveMap, die, rewards, useSpecialSkill

## nro.models.boss.cdrd
- CBoss.java (68) extends Boss: CBoss, joinMap, leaveMap, notifyPlayeKill
- Cadich.java (114) extends CBoss: Cadich, useSpecialSkill, rewards, idle, checkPlayerDie, changeToAttack, initTalk, getHead, getBody, getLeg, update, leaveMap
- Nadic.java (68) extends CBoss: Nadic, useSpecialSkill, rewards, idle, checkPlayerDie, changeToAttack, initTalk
- Saibamen.java (100) extends CBoss: Saibamen, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, injured, selfExplosion

## nro.models.boss.cell
- SieuBoHung.java (112) extends FutureBoss: SieuBoHung, useSpecialSkill, attack, leaveMap, joinMap, idle, rewards, checkPlayerDie, initTalk
- XenBoHung.java (160) extends FutureBoss: XenBoHung, useSpecialSkill, attack, injured, tuSat, joinMap, idle, rewards, checkPlayerDie, initTalk, leaveMap
- XenBoHung1.java (101) extends FutureBoss: XenBoHung1, useSpecialSkill, rewards, attack, idle, checkPlayerDie, initTalk, leaveMap
- XenBoHung2.java (99) extends FutureBoss: XenBoHung2, useSpecialSkill, rewards, attack, idle, checkPlayerDie, initTalk, leaveMap
- XenBoHungHoanThien.java (101) extends FutureBoss: XenBoHungHoanThien, useSpecialSkill, rewards, attack, idle, checkPlayerDie, initTalk, joinMap, leaveMap
- XenCon.java (107) extends FutureBoss: XenCon, useSpecialSkill, rewards, attack, idle, leaveMap, checkPlayerDie, joinMap, initTalk
- XenMax.java (98) extends FutureBoss: XenMax, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.chill
- Chill.java (72) extends FutureBoss: Chill, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Chill2.java (68) extends FutureBoss: Chill2, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.cold
- Cooler.java (74) extends FutureBoss: Cooler, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Cooler2.java (65) extends FutureBoss: Cooler2, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.dhvt
- BossDHVT.java (135) extends Boss: BossDHVT, useSpecialSkill, rewards, initTalk, idle, checkPlayerDie, attack, changeToAttack, joinMap, immortalMp, update, notifyPlayeKill, leaveMap
- ChaPa.java (16) extends BossDHVT: ChaPa
- ChanXu.java (16) extends BossDHVT: ChanXu
- JackyChun.java (16) extends BossDHVT: JackyChun
- LiuLiu.java (16) extends BossDHVT: LiuLiu
- ODo.java (17) extends BossDHVT: ODo
- PonPut.java (16) extends BossDHVT: PonPut
- SoiHecQuyn.java (16) extends BossDHVT: SoiHecQuyn
- TauPayPay.java (16) extends BossDHVT: TauPayPay
- ThienXinHang.java (54) extends BossDHVT: ThienXinHang, useSpecialSkill, attack, phanThan
- ThienXinHangClone.java (64) extends BossDHVT: ThienXinHangClone, update
- Xinbato.java (16) extends BossDHVT: Xinbato
- Yamcha.java (16) extends BossDHVT: Yamcha

## nro.models.boss.event
- Beetle.java (85) extends EscortedBoss: Beetle, injured, useSpecialSkill, rewards, initTalk, idle, setEscort, checkPlayerDie, notifyPlayeKill, joinMap, joinMapEscort, die, leaveMap
- EscortedBoss.java (86) extends Boss: EscortedBoss, attack, move, joinMapEscort, setEscort, stopEscorting
- NightLord.java (82) extends EscortedBoss: NightLord, injured, useSpecialSkill, rewards, initTalk, idle, checkPlayerDie, notifyPlayeKill, joinMap, joinMapEscort, die, leaveMap

## nro.models.boss.fide
- FideDaiCa1.java (64) extends FutureBoss: FideDaiCa1, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- FideDaiCa2.java (67) extends FutureBoss: FideDaiCa2, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap, joinMap
- FideDaiCa3.java (67) extends FutureBoss: FideDaiCa3, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap, joinMap
- FideGold.java (64) extends Boss: FideGold, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.iboss
- BossInterface.java (27) extends IBossStatus: 
- IBossInit.java (17) extends IBossOutfit: 
- IBossOutfit.java (18): 
- IBossStatus.java (23) extends IBossInit: 

## nro.models.boss.mabu_war
- BossMabuWar.java (57) extends Boss: BossMabuWar, useSpecialSkill, rewards, initTalk, idle, checkPlayerDie
- BuiBui_Tang2.java (101) extends BossMabuWar: BuiBui_Tang2, idle, injured, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, leaveMap
- BuiBui_Tang3.java (103) extends BossMabuWar: BuiBui_Tang3, idle, injured, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, leaveMap
- Calich_Tang5.java (107) extends BossMabuWar: Calich_Tang5, idle, injured, changeToAttack, rewards, checkPlayerDie, initTalk, joinMap, getMapCanJoin, leaveMap
- Drabula_Tang1.java (108) extends BossMabuWar: Drabula_Tang1, idle, injured, changeToAttack, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, leaveMap
- Drabula_Tang5.java (170) extends BossMabuWar: Drabula_Tang5, attack, idle, injured, changeToAttack, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, talk, leaveMap
- Drabula_Tang6.java (150) extends BossMabuWar: Drabula_Tang6, idle, attack, injured, die, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, leaveMap
- Goku_Tang5.java (107) extends BossMabuWar: Goku_Tang5, idle, injured, changeToAttack, rewards, checkPlayerDie, joinMap, getMapCanJoin, initTalk, leaveMap
- Mabu_14H.java (162) extends BossMabuWar: Mabu_14H, attack, injured, joinMap, getMapCanJoin, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Mabu_Tang6.java (124) extends BossMabuWar: Mabu_Tang6, attack, injured, joinMap, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SuperBu_14H.java (135) extends BossMabuWar: SuperBu_14H, attack, injured, joinMap, getMapCanJoin, idle, rewards, checkPlayerDie, initTalk, leaveMap
- Yacon_Tang4.java (134) extends BossMabuWar: Yacon_Tang4, attack, injured, joinMap, getMapCanJoin, idle, rewards, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.nappa
- Kuku.java (63) extends FutureBoss: Kuku, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- MapDauDinh.java (54) extends FutureBoss: MapDauDinh, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk
- Rambo.java (55) extends FutureBoss: Rambo, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk

## nro.models.boss.robotsatthu
- Android19.java (88) extends Boss: Android19, joinMap, leaveMap, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, injured
- Android20.java (81) extends Boss: Android20, joinMap, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, injured
- KingKong.java (65) extends Boss: KingKong, joinMap, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk
- Pic.java (62) extends Boss: Pic, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Poc.java (65) extends Boss: Poc, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.tieudoisatthu
- So1.java (61) extends FutureBoss: So1, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- So2.java (59) extends FutureBoss: So2, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- So3.java (59) extends FutureBoss: So3, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- So4.java (57) extends FutureBoss: So4, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- TieuDoiTruong.java (77) extends FutureBoss: TieuDoiTruong, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap, joinMap

## nro.models.boss.traidat
- BULMA.java (100) extends Boss: BULMA, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- CHICHITHO.java (100) extends Boss: CHICHITHO, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap
- POCTHO.java (101) extends Boss: POCTHO, useSpecialSkill, attack, rewards, idle, checkPlayerDie, initTalk, leaveMap

## nro.models.boss.tramtau
- Champa.java (66) extends Boss: Champa, useSpecialSkill, rewards, idle, checkPlayerDie, initTalk, leaveMap
- Vados.java (84) extends Boss: Vados, useSpecialSkill, rewards, idle, leaveMap, checkPlayerDie, initTalk

## nro.models.boss.vip
- BabyVegeta.java (103) extends Boss: BabyVegeta, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- CadicSuperSaiyan.java (103) extends Boss: CadicSuperSaiyan, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- DraburaFrost.java (102) extends Boss: DraburaFrost, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- EvilBuu.java (103) extends Boss: EvilBuu, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- GohanZombie.java (103) extends Boss: GohanZombie, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- GokuSuperSaiyan.java (102) extends Boss: GokuSuperSaiyan, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- HeartsGold.java (103) extends Boss: HeartsGold, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- JirenCuongNo.java (102) extends Boss: JirenCuongNo, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- RongDen1Sao.java (101) extends Boss: RongDen1Sao, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SaiyanGodTrunks.java (101) extends Boss: SaiyanGodTrunks, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SuperBrolyHuyenThoai.java (102) extends Boss: SuperBrolyHuyenThoai, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- SuperZamasuWhite.java (103) extends Boss: SuperZamasuWhite, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- ToppoGOD.java (103) extends Boss: ToppoGOD, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- VegetaHakai.java (103) extends Boss: VegetaHakai, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap
- ZamasuZombie.java (103) extends Boss: ZamasuZombie, useSpecialSkill, attack, idle, rewards, checkPlayerDie, initTalk, leaveMap

## nro.models.clan
- Buff.java (9): 
- Clan.java (415): Clan, initialize, getLeader, getRole, isLeader, isDeputy, addSMTNClan, sendMessageClan, addClanMessage, getClanMessage, getCurrClanMessages, sendRemoveClanForAllMember, sendMyClanForAllMember, sendRemoveForAllMember, sendFlagBagForAllMember, addMemberOnline, removeMemberOnline, getPlayerOnline, addClanMember, removeClanMember, getCurrMembers, getMembers …+4
- ClanMember.java (76): ClanMember, getNumDateFromJoinTimeToToday
- ClanMessage.java (47): ClanMessage

## nro.models.consignment
- ConsignmentItem.java (22) extends Item: 
- ConsignmentService.java (16): 
- ConsignmentShop.java (467): getInstance, handler, upTop, cancelConsign, addItem, addExpiredItem, removeItem, consign, findItemConsign, getItemConsignByTab, getItemCanConsign, canConsign, getTabByType, buy, getMoney, nextPage, show, showItemCanConsign, getDaysExpried, showExpiringItems, sendExpirationNotification

## nro.models.dragon
- AbsDragon.java (65): AbsDragon, sendNotify, run
- Poruga.java (114) extends AbsDragon: Poruga, openMenu, summon, reSummon, showWishes, run, callDragon, leave

## nro.models.intrinsic
- Intrinsic.java (52): Intrinsic, getDescription, getName
- IntrinsicPlayer.java (30): IntrinsicPlayer, dispose

## nro.models.item
- CaiTrang.java (28): CaiTrang, getID
- FlagBag.java (20): 
- HeadAvatar.java (22): HeadAvatar
- Item.java (329): isNotNullItem, Item, getInfo, getInfoItem, getDisplayOptions, getContent, canConsign, isSKH, isKimDan, isTrangBiNangCap, isKhoa, isDaCuongHoa, isDa, isDTS, isDTL, isTrangBiThuHoi, isThanhTon, isThanLongThach, isCongThuc, isSachCoNgu, isDHD, isManhTS …+12
- ItemLuckyRound.java (24): ItemLuckyRound
- ItemOption.java (90): ItemOption, getOptionString, format, dispose
- ItemOptionLuckyRound.java (23): ItemOptionLuckyRound
- ItemOptionTemplate.java (28): ItemOptionTemplate
- ItemReward.java (26): toString
- ItemTemplate.java (33): 
- ItemTime.java (181): ItemTime, update, dispose
- ItemTimeSieuCap.java (133): ItemTimeSieuCap, update, dispose
- MinipetTemplate.java (16): 

## nro.models.map
- EffectEventManager.java (71): gI, load, add
- EffectEventTemplate.java (21): 
- EffectMap.java (15): 
- ItemMap.java (116): ItemMap, update, reAppearItem, dispose
- Map.java (291): Map, addZone, addNpc, initZone, initReferee, initNpc, run, initMob, initTrapMap, initItem, getNpc, yPhysicInTop, isTileTop, isMapLang, removeZone
- MapTemplate.java (52): MapTemplate, isMapOffline
- NamekBall.java (50) extends ItemMap: NamekBall, update, setZone, reAppearItem
- SantaCity.java (182) extends Map: SantaCity, initZone, random, enter, leave, open, timer, stopTimers, reset, close
- Satellite.java (74) extends ItemMap: Satellite, getPlayersSameClan, update
- SatelliteDefense.java (37) extends Satellite: SatelliteDefense, buff
- SatelliteExp.java (37) extends Satellite: SatelliteExp, buff
- SatelliteHP.java (31) extends Satellite: SatelliteHP, buff
- SatelliteMP.java (32) extends Satellite: SatelliteMP, buff
- TrapMap.java (38): doPlayer
- WayPoint.java (31): 
- Zone.java (1103): getRandomMobInMap, Zone, isExcludedId, isFullPlayer, addMob, updateMob, getTotalHP, updatePlayer, updateReferee, updateTestDame, updateItem, update, getNumOfPlayers, getNumOfBosses, isBossCanJoin, getNotBosses, getPlayers, getHumanoids, getBosses, getPlayersCadic, getPlayersFide, addPlayersCadic …+32

## nro.models.map.challenge
- MartialCongress.java (197): update, ready, toTheNextRound, resetSkill, timeOut, champion, leave, reward, endChallenge
- MartialCongressManager.java (53): gI, update, add, remove
- MartialCongressService.java (77): gI, startChallenge, moveFast, sendTypePK, getMapChalllenge

## nro.models.map.dungeon
- Dungeon.java (128): Dungeon, addZone, removeZone, find, update, setTime, close, sendNotification, sendMessage
- DungeonManager.java (93): DungeonManager, generateID, addDungeon, removeDungeon, find, update, start, shutdown, run
- SnakeRoad.java (203) extends Dungeon: SnakeRoad, init, addBoss, removeBoss, getBoss, initBoss, update, join, finish

## nro.models.map.dungeon.zones
- ZDungeon.java (101) extends Zone: ZDungeon, init, setTextTime, enter, changeMapWaypoint, close
- ZSnakeRoad.java (129) extends ZDungeon: ZSnakeRoad, isKilledAll, changeMapWaypoint, enter, initMob, playerMove, close

## nro.models.map.mabu
- MabuWar.java (195): gI, setTime, isTimeMabuWar, update, kickOutOfMap, joinMapMabuWar, removeAllBoss, sendMenuGotoNextFloorMabuWar, sendMenuFinishMabuWar, BabidayTalk, getMapLastFloor, initMabu
- MabuWar14h.java (109): gI, setTime, isTimeMabuWar, update, kickOutOfMap, removeAllBoss

## nro.models.map.phoban
- BanDoKhoBau.java (153): BanDoKhoBau, update, openBanDoKhoBau, resetBanDo, initBoss, finish, getMapById, addZone, sendTextBanDoKhoBau
- DoanhTrai.java (166): DoanhTrai, update, openDoanhTrai, finish, resetDoanhTrai, getMapById, initBoss, sendTextDoanhTraiAllClan, addZone

## nro.models.map.war
- BlackBallWar.java (300): BlackBallWar, gI, setTime, addMap, dropBlackBall, update, win, kickOutOfMap, changeMap, joinMapBlackBallWar, pickBlackBall, xHPKI
- NamekBallWar.java (107): gI, pickBall, summonDragon, dropBall

## nro.models.map.zones
- Barrack.java (84) extends Zone: Barrack, addPlayer, adjustMonsterAttributes, getMaxPoint

## nro.models.mob
- BigBoss.java (62) extends Mob: BigBoss, move, getPlayerCanAttack, attackPlayer
- GuardRobot.java (84) extends BigBoss: GuardRobot, attack, setDie, send, hide
- Hirudegarn.java (223) extends BigBoss: Hirudegarn, getSys, getListPlayerCanAttack, jump, send, attackPlayer, attack, transform, flyTo, move, setDie, injured
- IBigBoss.java (21): 
- Mob.java (263): Mob, getSys, setTiemNang, isDie, injured, getTiemNangForPlayer, update, attackPlayer, getPlayerCanAttack, addPlayerAttack, setDie
- MobEffectSkill.java (194): MobEffectSkill, update, isHaveEffectSkill, startStun, removeStun, setThoiMien, removeThoiMien, setStartBlindDCTT, removeBlindDCTT, setTroi, removeAnTroi, removeSocola, removeBinh, setSocola, setBinh
- MobFactory.java (28): newMob
- MobMe.java (141) extends Mob: MobMe, calPercent, update, attack, spawn, goToMap, removeMobInMap, mobMeDie, dispose
- MobPoint.java (54): MobPoint, getHpFull, setHpFull, getHP, setHP, getDameAttack
- MobReward.java (33): MobReward
- MobTemplate.java (23): 
- Octopus.java (132) extends BigBoss: Octopus, getPlayerCanAttack, move, hide, send, moveX, attack, setDie

## nro.models.npc
- BaseMenu.java (36): openMenu
- IAtionNpc.java (18): 
- Npc.java (200): Npc, initBaseMenu, createOtherMenu, openBaseMenu, npcChat, openShopWithGender, openShopWithGender2, openShop, canOpenNpc
- NpcFactory.java (8130): NpcFactory, createNPC, openBaseMenu, confirmMenu, changeMap, createNpcRongThieng, createNpcConMeo, openMenuSuKien, getMenuSuKien, getMenuLamBanh, safeMultiply, calPercent
- NpcManager.java (60): getByIdAndMap, getNpc, getNpcsByMapPlayer
- NpcTemplate.java (19): 

## nro.models.npc.specialnpc
- MabuEgg.java (108): MabuEgg, createMabuEgg, sendMabuEgg, getSecondDone, openEgg, destroyEgg, subTimeDone, dispose
- MagicTree.java (285): MagicTree, update, loadMagicTree, openMenuTree, harvestPea, showConfirmUpgradeMagicTree, showConfirmUnuppgradeMagicTree, upgradeMagicTree, unupgradeMagicTree, fastRespawnPea, fastUpgradeMagicTree, getMaxPea, getSecondPerPea, getSecondPea, getSecondUpgrade, getTextMenuUpgrade, getTimeUpgrade, dispose

## nro.models.phuban.DragonNamecWar
- TranhNgoc.java (141): gI, getPlayersCadic, getPlayersFide, addPlayersCadic, addPlayersFide, removePlayersCadic, removePlayersFide, setTime, update, kickOutOfMap, isTimeRegisterWar, isTimeStartWar
- TranhNgocService.java (197): getInstance, sendCreatePhoBan, sendUpdateLift, sendEndPhoBan, sendUpdateTime, sendUpdatePoint, givePrice, pickBall, dropBall

## nro.models.player
- Charms.java (106): Charms, addTimeCharms, dispose
- EffectFlagBag.java (49): EffectFlagBag, reset, dispose
- EffectSkill.java (204): EffectSkill, removeSkillEffectWhenDie, update, isHaveEffectSkill, dispose
- EffectSkin.java (562): EffectSkin, update, updateYacon, setxDameChuong, updateBuiBui, updateMabu, updateDrabula, updateNezuko, updateTanjiro, updateInoHashi, updateInosuke, updateZenitsu, updateCTHaiTac, updateXenHutXungQuanh, updateOdo, updateTrainArmor, sendPlayerPrepareBom, updateVoHinh, dispose
- Enemy.java (12) extends Friend: 
- Friend.java (21): 
- Fusion.java (35): Fusion, update, dispose
- Gift.java (25): Gift, dispose
- IDMark.java (78): getTranhNgoc, setTranhNgoc, setTypeLuckyRound, getTypeLuckyRound, getIndexMenu, setIndexMenu, isBaseMenu, setTypeInput, getTypeInput, setShopId, getShopId, getTypeChangeMap, setTypeChangeMap
- Inventory.java (141): Inventory, getGem, getGold, getGoldLimit, getGoldDisplay, getParam, haveOption, getRuby, subGem, subGold, subRuby, addGold, dispose
- ListFriendEnemy.java (20): ListFriendEnemy
- Location.java (14): 
- MiniPet.java (163) extends Player: callMiniPet, setPart, version, getHead, getBody, getLeg, MiniPet, changeStatus, reCall, joinMapMaster, moveIdle, update, followMaster
- NPoint.java (3186): NPoint, initPowerLimit, calPoint, setPoint, setPointWhenWearClothes, setChisoPorata, setDameTrainArmor, setNeDon, setHpHoi, setMpHoi, setHpMax, setHp, setMpMax, setMp, setDame, setDef, setCrit, setCritDame, setSpeed, resetPoint, addHp, addMp …+35
- Pet.java (1029) extends Player: getStatus, version, Pet, changeStatus, joinMapMaster, goHome, getTextStatus, fusion, fusion2, fusion3, fusion4, fusion5, fusion6, unFusion, fusionEffect, exitMapFusion, moveIdle, update, askPea, useSkill3, useSkill4, increasePoint …+13
- PetFollow.java (23): 
- Player.java (1526): Player, getPowerPoint, getPetFollow, setPetFollow, addPowerPoint, getPercentPowerPont, addPercentPowerPoint, resetPowerPoint, setUseSpaceShip, getBodyItemByType, getUseSpaceShip, isDie, isAlive, checkdanhhieutop1, checkdanhhieutop2, checkdanhhieutop3, check99ThucAnHuyDiet, setSession, sendMessage, getSession, version, isVersionAbove …+29
- PlayerEvent.java (54): PlayerEvent, setTimeCookChungCake, setTimeCookTetCake, addEventPoint, subEventPoint, update
- Referee.java (97) extends Player: initReferee, getHead, getBody, getLeg, joinMap, version, update, init
- RewardBlackBall.java (96): RewardBlackBall, reward, getRewardSelect, getReward, dispose
- SetClothes.java (327): SetClothes, setup, setupSKT, setupAN, setupLongAn, setupDTS, setupDHD, setupDTL, setDefault, dispose
- SkillSpecial.java (130): SkillSpecial, update, setSkillSpecial, closeSkillSpecial, close, start, dispose
- TestDame.java (168) extends Player: getSpawnX, getSpawnY, initTestDame, getHead, getBody, getLeg, version, update, init, isTargetMap, spawnOneInZone

## nro.models.pvp
- ChallengePVP.java (58) extends PVP: ChallengePVP, start, sendResultMatch, reward
- PVP.java (68): start, update, finishPVP, dispose
- RevengePVP.java (67) extends PVP: RevengePVP, update, sendResultMatch, reward
- martialCongressPVP.java (41) extends PVP: martialCongressPVP, sendResultMatch, reward

## nro.models.shop
- ItemShop.java (54): ItemShop
- Shop.java (47): Shop
- TabShop.java (50): TabShop

## nro.models.skill
- Hit.java (34): Hit, addTarget, addHit
- NClass.java (40): getSkillTemplate, getSkillTemplateByName
- PlayerSkill.java (148): PlayerSkill, scheduleOnce, scheduleFixedRate, safe, getSkillbyId, sendSkillShortCut, sendSkillShortCutNew, getIndexSkillSelect, getSizeSkill, dispose
- Skill.java (102): Skill
- SkillNotFocus.java (26) extends Skill: SkillNotFocus
- SkillTemplate.java (40): 

## nro.models.task
- Achivement.java (50): isDone, getCount, getMaxCount
- AchivementTemplate.java (25): 
- SideTask.java (91): reset, SideTask, isDone, getName, getLevel, getPercentProcess
- SideTaskTemplate.java (20): SideTaskTemplate
- SubTaskMain.java (36): SubTaskMain
- TaskMain.java (40): TaskMain
- TaskPlayer.java (36): TaskPlayer, dispose

## nro.netty
- HttpRequestHandler.java (327) extends SimpleChannelInboundHandler: 
- HttpServerInitializer.java (18) extends ChannelInitializer: 
- NettyServer.java (61) extends Thread: 

## nro.noti
- Alert.java (17): 
- NotiManager.java (97): getInstance, load, loadNoti, loadAlert, addNoti, sendAlert, sendNoti
- Notification.java (30): 

## nro.power
- Caption.java (36): getCaption
- CaptionManager.java (99): getInstance, CaptionManager, load, add, remove, find, getLevel
- PowerLimit.java (30): 
- PowerLimitManager.java (81): getInstance, PowerLimitManager, load, add, remove, get

## nro.resources
- AbsResources.java (253): AbsResources, init, initEffectData, initMobData, initIBN, initDataVersion, initBGSmallVersion, initSmallVersion, setFolder, readAllBytes, readAllLines, readString, getRawIconData, getRawBGData, getRawIBNData, getRawMobData, getRawEffectData, putData, getData, getIBN, getMobData, getEffectData
- RNormal.java (18) extends AbsResources: RNormal
- RSpecial.java (18) extends AbsResources: RSpecial
- Resources.java (381): getInstance, Resources, init, readAllBytes, readAllLines, find, downloadResources, sendNumberOfFiles, fileTransferCompleted, sendResVersion, fileTransfer, downloadIconData, downloadBGTemplate, sendSmallVersion, sendBGVersion, downloadIBN, requestMobTemplate, loadMoData, sendEffectTemplate, effData, loadEffectData

## nro.resources.entity
- EffectData.java (69): getData
- Frame.java (24): 
- ImageByName.java (24): 
- MobData.java (73): setData
- Sprite.java (24): 

## nro.sendEff
- SendEffect.java (556): getInstance, sendChanThienTu, sendChanThienTuAll, sendDanhHieu, sendDanhHieuAll, createMessage2, createMessage, removeTitle, SendEffDanhHieu, Name_Danh_Hieu, send_danh_hieu, SendThreadEffDanhHieu, update

## nro.server
- AutoMaintenance.java (101): start, run, execute, openCmd
- Client.java (360): Client, getPlayers, gI, put, remove, clear, createBot, dispose, kickSession, getPlayer, getPlayerByUser, getSession, close, update, run, show
- Controller.java (1005): getInstance, hexToBytes, onMessage, messageNotLogin, sendPreloginCase5, handleRegisterWithCaptcha, messageNotMap, messageSubCommand, createChar, login2, sendInfo
- FlatDarculaLaf.java (14): 
- GameConfig.java (39): GameConfig, load
- Maintenance.java (55) extends Thread: Maintenance, gI, start, run
- Manager.java (1247): run, shutdown, gI, Manager, initRandomItem, initMap, loadDatabase, getMapTemplate, loadEventCount, updateEventCount, loadAttributeServer, removeClan, updateAttributeServer, loadProperties, readTileIndexTileType, readTileMap, getClanById, addClan, getNumClan, getCaiTrangByItemId, getMobTemplateByTemp
- MenuController.java (62): getInstance, openMenuNPC, doSelectMenu
- ServerLog.java (59): logCombine, logItemDrop, toTimeString
- ServerLogSavePlayer.java (65): ServerLogSavePlayer, gI, run, add
- ServerManager.java (339): init, gI, main, run, activeLogin, activeServerSocket, canConnectWithIp, disconnect, activeCommandLine, activeGame, close, saveAll, autoTask
- ServerNotify.java (66) extends Thread: ServerNotify, gI, run, sendThongBaoBenDuoi, notify
- panel.java (308) extends JPanel: panel, buildStatusGroup, label, valueLabel, buildControlsGroup, btn, btnAccent, updateStatusNow, bytes, actionPerformed, removeNotify, ExpInputDialog, main

## nro.server.io
- Message.java (77): Message, writer, reader, getData, transformData, cleanup, dispose
- MessageCollector.java (99): MessageCollector, run, readMessage, readKey, close
- MessageSender.java (143): getNumMessage, MessageSender, addMessage, run, doSendMessage, writeKey, sendSessionKey, close
- Session.java (383): update, initItemsReward, getNumOfMessages, Session, sendMessage, doSendMessage, disconnect, setClientType, isVersionAbove, getName, sendSessionKey, canConnectWithIp, login, finishUpdate, enter, loadPlayer

## nro.server.io.pool
- SendPool.java (34): SendPool, gI, send
- SendTask.java (39): SendTask, run

## nro.server.model
- AntiLogin.java (53): canLogin, wrong, reset, getNotifyCannotLogin

## nro.services
- AutoBotChatService.java (93): gI, run
- BanDoKhoBauService.java (93): BanDoKhoBauService, gI, getMapBanDoKhoBau, openBanDoKhoBau
- BangTin.java (78): gI, Send_BangTin, load_BangTin
- BossFollowerService.java (53): start, run
- BotAIService.java (123): updateBot, tryChangeZoneOrMap, tryJumpMap, getSafeZoneForBot
- BotManager.java (277): gI, createMultipleBots, createBot, getBots, safeSetCostume, getSafeStartZone, randomFusionForBot
- CaptchaService.java (132): Entry, Challenge, generate, verify, randomCode, render, toPNG, sha256
- ChatGlobalService.java (223): ChatGlobalService, gI, chat, run, chatGlobal, transformText, ChatGlobal
- ClanService.java (1153): ClanService, gI, getClanById, isNullClan, getClans, getClan, clanMessage, clanDonate, joinClan, clanRemote, clanInvite, sendInviteClan, acceptJoinClan, acceptAskJoinClan, cancelAskJoinClan, askForPea, askForJoinClan, changeInfoClan, createClan, sendListClan, sendListMemberClan, sendRemoveClan …+17
- DoanhTraiService.java (83): DoanhTraiService, gI, getMapDoanhTrai, openDoanhTrai
- EffSkinService.java (156): gI, setInvisible, removeInvisible, setSlow, removeSlow, setSocola, removeSocola, setHoaDa, removeHoaDa, setCuongNo, removeCuongNo, setInosuke, removeInosuke, setTanjiro, removeTanjiro, setZenitsu, removeZenitsu, setInoHashi, removeInoHashi, sendEffectPlayer
- EffectSkillService.java (541): EffectSkillService, gI, sendEffectUseSkill, sendEffectPlayer, sendEffectMob, removeUseTroi, removeAnTroi, setAnTroi, setUseTroi, setThoiMien, removeThoiMien, startStun, removeStun, setSocola, setBinh, removeSocola, removeBinh, sendMobToSocola, sendMobToBinh, setBlindDCTT, removeBlindDCTT, setStartHuytSao …+21
- FlagBagService.java (125): gI, sendIconFlagChoose, sendIconEffectFlag, sendListFlagClan, getFlagBag, getFlagBagByName, getFlagsForChooseClan
- FriendAndEnemyService.java (334): gI, controllerFriend, controllerEnemy, reloadFriend, reloadEnemy, openListFriend, openListEnemy, makeFriend, removeFriend, removeEnemy, chatPrivate, acceptMakeFriend, goToPlayerWithYardrat, addEnemy
- GameDuDoan.java (408): gI, addPlayerXiu, addPlayerTai, setTime, isTimeOpen, run, Send_TaiXiu, thongbao, DatTai, DatXiu, chat
- GiftService.java (190): GiftService, gI, use, isUsedGiftCode, addUsedGiftCode
- GiftcodePlayer.java (194): GiftcodePlayer, gI, use, isUsedGiftCode, addUsedGiftCode
- IntrinsicService.java (172): gI, getIntrinsics, getIntrinsicById, sendInfoIntrinsic, showAllIntrinsic, sattd, satnm, setxd, showMenu, showConfirmOpen, showConfirmOpenVip, changeIntrinsic, open, openVip
- InventoryService.java (1226): gI, copyItemsBag, copyList, existItemBag, existItemInList, hasOptionTemplateId, getIndexBag, addItemBag, addItemBox, addItemBody, addItemNotUpToUpQuantity, addItemList, isItemIncremental, isItemIncrementalOption, throwItem, arrangeItems, putItemBag, putItemBox, putItemBody, itemBagToBody, itemBodyToBag, itemBagToPetBody …+52
- ItemMapService.java (67): gI, pickItem, removeItemMapAndSendClient, sendItemMapDisappear, removeItemMap, isBlackBall, isNamecBall
- ItemService.java (767): gI, createItemNull, createItemFromItemShop, copyItem, convertToConsignmentItem, createNewItem, TaoMoiVatPhamVaoTuiDo, DoThienSu, settaiyoken, setgenki, setkamejoko, setgodki, setgoddam, setsummon, setgodgalick, setmonkey, setgodhp, otpts, randomSKHId, itemSKH, getListOptionItemShop, createItemSetKichHoat …+12
- ItemTimeService.java (246): gI, sendAllItemTime, turnOnTDLT, turnOffTDLT, sendCanAutoPlay, sendTextRongBang, sendTextRongSieuCap, sendTextDoanhTrai, sendTextBanDoKhoBau, removeTextDoanhTrai, removeTextTime, sendTextTime, sendItemTime, removeItemTime, sendItemTimeBienHinhSc
- KhamNgoc.java (215): gI, loadKhamNgoc, Send_KhamNgocTemplate, Send_KhamNgoc_Player, activeKhamNgoc, NangCapKhamNgoc
- KhamNgocPlayer.java (18) extends Player: 
- KhamNgocTemplate.java (17): 
- MapService.java (374): gI, getWaypointPlayerIn, readTileIndexTileType, readTileMap, getMapCanJoin, getMapById, getMapForCalich, getZoneJoinByMapIdAndZoneId, getMapWithRandZone, getPlanetName, getMapCapsule, getMapBlackBall, addListMapCapsule, sendPlayerMove, goToMap, exitMap, getZoneForBot, isMapOffline, isMapBlackBallWar, isMapMabuWar14H, isMapMabuWar, isMapKhiGas …+12
- MobService.java (252): MobService, gI, sendMobStillAliveAffterAttacked, sendMobDieAffterAttacked, hutItem, mobReward, mobAttackPlayer, sendMobAttackMe, sendMobAttackPlayer, hoiSinhMob, hoiSinhMobDoanhTrai, initMobDoanhTrai, initMobBanDoKhoBau, main, dropItemTask, isMonterFly
- NpcService.java (86): gI, createMenuRongThieng, createMenuConMeo, createMenu, createTutorial, getAvatar
- OpenPowerService.java (81): OpenPowerService, gI, openPowerBasic, openPowerSpeed
- PetService.java (1111): gI, createNormalPet, createMabuPet, createBerusPet, createPicPet, createKaidoPet, createItachiPet, createAndroidPet, createNgoKoPet, createSoi3DauPet, createDaiThanhPet, createPetNew2, createPetNew3, createPetNew4, createPetNew5, createPetNew6, createPetNew7, createPetNew8, changeNormalPet, changeMabuPet, changeBerusPet, changePicPet …+36
- PhongThiNghiem.java (259): gI, thoiGianDieuChe, loadPhongThiNghiem, Send_PhongThiNghiem_Template, Send_PhongThiNghiem_Player, dieu_che, nhan_item, tangTocPtn, huyPtn, mo_rong
- PhongThiNghiem_Player.java (18) extends Player: 
- PhongThiNghiem_Template.java (16): 
- PhucLoi.java (310): gI, Send_PhucLoi, Check_active, countPlayer, Active_PhucLoi, load_PhucLoi, load_PhucLoiTab
- PhucLoiManager.java (18): 
- PlayerService.java (382): PlayerService, gI, dailyLogin, checkNewWeek, sendTNSM, sendMessageAllPlayer, sendMessageIgnore, sendInfoHp, sendInfoMp, sendInfoHpMp, hoiPhuc, sendInfoHpMpMoney, playerMove, sendCurrentStamina, sendMaxStamina, changeAndSendTypePK, changeTypePK, sendTypePk, banPlayer, hoiSinh, createPlayer, savePlayer …+3
- RewardService.java (1327): RewardService, gI, getMobReward, getRewardItems, initQuantityGold, initBaseOptionClothes, initBaseOptionSaoPhaLe, initBasOptionDaNangCap, initStarOption, initEventOption, initExpiryDateOption, initNotTradeOption, initDepositOption, initActivationOption, addOptionSkhMap, getMaxStarOfItemReward, getListItemLuckyRound, RatioStar, rewardFirstTimeLoginPerDay
- RuongSuuTam.java (380): gI, loadRuongSuuTam, Send_RuongSuuTamTemplate, SendAllRuong, Send_RuongCaiTrang, Send_RuongPhuKien, Send_RuongPet, Send_RuongLinhThu, Send_RuongThuCuoi, mangItem, thaoItem, moRongRuong, nameRuong, activeRuongSuuTam
- RuongSuuTamPlayer.java (20): 
- RuongSuuTamTemplate.java (16): 
- Service.java (2390): getInstance, sendMessAllPlayer, sendMessAllPlayerIgnoreMe, sendMessAllPlayerInMap, sendMessAnotherNotMeInMap, Send_Info_NV, sendInfoPlayerEatPea, sendPopUpMultiLine, loginDe, resetPoint, clearMap, chat, chatMap, infoall, MenuDanhHieu, BuffDanhHieu, capVIP, chatJustForMe, point, player, messageNotLogin, messageNotMap …+73
- SkillService.java (1444): SkillService, gI, useSkill, upgradeSkillSpecial, userSkillSpecial, useSkillAttack, useSkillAlone, useSkillBuffToPlayer, phanSatThuong, hutHPMP, playerAttackPlayer, playerAttackMob, sendPlayerPrepareSkill, sendPlayerPrepareBom, canUseSkillWithMana, canUseSkillWithCooldown, affterUseSkill, setMpAffterUseSkill, setLastTimeUseSkill, canHsPlayer, canAttackPlayer, sendPlayerAttackMob …+10
- SubMenuService.java (87): SubMenuService, gI, controller, showMenuForAdmin, showSubMenu, SubMenu
- TamBao.java (637): gI, TamBao, loadItem_TamBao, padWithFallback, SpinPool, ensureSpinPool14, sanitizePercent, pickIndexByPercent, QuayTamBao, QuayTamBaoWithKey, buildFallbackPool, Send_QuayThuong, Send_TamBao, Send_MocTamBao, Active_TamBao, Check_active, load_mocTamBao, getNullableInt, addOptionsFromCompact, parseItemObject, addOptionFromFlexibleEntry, toInt …+3
- TamBao_Item.java (12) extends Item: 
- TaskService.java (3128): gI, getTaskMainById, sendTaskMain, sendNextTaskMain, sendUpdateCountSubTask, sendNextSubTask, sendInfoCurrentTask, checkDoneTaskTalkNpc, checkDoneTaskJoinClan, checkDoneTaskGetItemBox, checkDoneTaskPower, checkDoneTaskUseTiemNang, checkDoneTaskGoToMap, checkDoneTaskPickItem, checkDoneTaskMakeFriend, checkDoneTaskConfirmMenuNpc, checkDoneTaskKillBoss, checkDoneTaskKillMob, doneTask, npcSay, rewardDoneTask, addDoneSubTask …+16
- TopService.java (530): TopService, gI, showTopSd, showTopSucManh, showTopHP, showTopKi, showTopNhiemVu, showTopVnd, sendTop, rowForPlayerOnline, rowForPlayerMaybeOffline, nonNull, notifyFail, Row, limitOnlineTop

## nro.services.func
- ChangeMapService.java (952): ChangeMapService, gI, openChangeMapTab, getLeastPopulatedZone, startAutoZoneChange, changeZone, changeMapBySpaceShip, changeMapNonSpaceship, changeMapInYard, changeMap, changeMapYardrat, getZoneTranhNgoc, finishLoadMap, sendEffectMeToMap, sendEffectMapToMe, spaceShipArrive, goToTuongLai, goToDBKB, goToPrimaryForest, goToHanhTinhBangGia, goToDiaNguc, goToHallowen …+5
- CombineNew.java (55): CombineNew, setTypeCombine, clearItemCombine, clearParamCombine, dispose
- CombineServiceNew.java (5381): CombineServiceNew, gI, openTabCombine, showInfoCombine, startCombine, GetTrangBiKichHoathuydiet, GetTrangBiKichHoatthiensu, doiManhKichHoat, chuyenhoahuydiet, openThuHoiTrangBi, PhanRaDoTS, openSKHts, openDTS, openCheTaoThanhTon, openCheTaoSKHThanhTon, openSKHVaiTho, getItemLevel, getItemLevelRada, getItemLevelDaCuongHoa, NangCapDoSKH, tangPhamChatKimDan, NangCapKimDan …+63
- EffectMapService.java (99): EffectMapService, gI, sendEffEvent, sendEffectMapToPlayer, sendCharEffect, sendEffectMapToAllInMap
- Input.java (667): Input, gI, doInput, createForm, createFormChangePassword, createFormGiftCode, createGiftMember, createFormFindPlayer, createFormChangeName, createFormChooseLevelBDKB, createFormChooseLevelCDRD, createFormTangRuby, createFormAddItem, createFormChangeNameByItem, createFormDoiXuBac, createFormDoiXuVang, createFormDoiThoiVang, createFormBuffItemVip, createFormBuffDanhHieu, TAI_taixiu, XIU_taixiu, ChonSo …+4
- InventoryServiceNew.java (21): gI
- LuckyRoundService.java (70): gI, openCrackBallUI, readOpenBall
- PVPServcice.java (247): PVPServcice, gI, controller, openSelectGold, sendInvitePVP, acceptPVP, findPvp, removePVP, finishPVP, openSelectRevenge, acceptRevenge, run
- RadaService.java (194): RadaService, getInstance, controller, cardAction, useCard, viewCollectionBook, useItemCard, setCardLevel, setIDAuraEff
- ShopService.java (1098): gI, getItemShop, getShop, getShopBua, openShopWhisThienSu, openShopBua, openShopNormal, openShopSpecial, openShopType0, openShopType3, buyItemShopNormal, isLimitItem, getBuyLimit, openBoxItemLuckyRound, getItemSideBoxLuckyRound, openBoxItemReward, getItemSideBoxReward, buyItem, IdItemNhan, GiaBan, NameItemNhan, showConfirmSellItem …+1
- SoMayMan.java (115): gI, addPlayerSMM, addPlayerTrung, KetQuaWin, PhatThuong, ThuaCuoc, ThuongOff, run
- SummonDragon.java (891): SummonDragon, active, gI, openMenuSummonShenron, summonShenron, summonBlackShenron, summonIceShenron, reSummonShenron, sendWhishesShenron, activeShenron, activeDragonNew, checkShenronBall, sendNotifyShenronAppear, confirmWish, showConfirmShenron, reOpenShenronWishes, shenronLeave, activeNight, activeDay
- TaiXiu.java (208): gI, addPlayerXiu, addPlayerTai, removePlayerXiu, removePlayerTai, run
- Trade.java (441): Trade, openTabTrade, addItemTrade, removeItemTrade, isItemCannotTran, isMinipet, cancelTrade, closeTab, dispose, lockTran, acceptTrade, startTrade, sendNotifyTrade, update
- TransactionService.java (166): TransactionService, gI, controller, sendInviteTrade, cancelTrade, run
- UseItem.java (3179): UseItem, gI, getItem, doItem, useSatellite, useItem, addOptionSkhStt, addOptionSkhHd, addOptionSkhTL, addOptionSkhTS, openskhNomal, openskhVip, skhtd, skhnm, skhxd, Hopts, skh_thanh_ton, skh_than_linh, skh_huy_diet, Set_TraiDat_TL, Set_Namec_TL, Set_Xayda_TL …+64

## nro.services.func.lr
- AbsLuckyRound.java (108): payAndGetStarted, openUI, result, addItemToBox
- LuckyRoundGem.java (61) extends AbsLuckyRound: gI, LuckyRoundGem, reward, checkMoney, payWithMoney
- LuckyRoundGold.java (64) extends AbsLuckyRound: gI, LuckyRoundGold, reward, checkMoney, payWithMoney

## nro.utils
- FileIO.java (94): readFile, loadFile, writeFile
- FileUtils.java (64): writeFile, getFolderSize, cutPng, addPath
- Log.java (56): log, success, warning, error
- SkillUtil.java (446): createSkill, createEmptySkill, createSkillLevel0, isUseSkillDam, isUseSkillChuong, isUseSkillBoom, getTimeMonkey, getPercentDameMonkey, getPercentHpMonkey, getTimeStun, getTimeSocola, getTimeBinh, getTimeShield, getTimeTroi, getTimeDCTT, getTimeThoiMien, getRangeStun, getRangeBom, getRangeQCKK, getPercentHPHuytSao, getPercentTriThuong, getPercentCharge …+18
- TimeUtil.java (196): diffDate, isTimeNowInRangex, getCurrDay, getCurrHour, getCurrMin, getTimeLeft, getMinLeft, getSecondLeft, getTime, getTimeNow, getTimeBeforeCurrent, main, formatTime, getTimeAgo, secToTime
- Util.java (654): randomBossId, highlightsItem, numberToMoney, powerToStringnew, powerToString, formatBigDame, powerToString_Long, format, formatNew, setTimeout, getDistance, msToTime, getDistanceByDir, nextInt, getOne, currentTimeSec, replace, debug, main, DoubleGioihan, DoubleToInter, LongGioihan …+22
