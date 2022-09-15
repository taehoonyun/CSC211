# CSC211

Hello This is Taehoon

# Create Harvester

Game.spawns['Spawn1'].spawnCreep([WORK,CARRY,MOVE],'Harvester1');

# role Harvester

var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
        else {
            var targets = creep.room.find(FIND_STRUCTURES, {
                    filter: (structure) => {
                        return (structure.structureType == STRUCTURE_EXTENSION || structure.structureType == STRUCTURE_SPAWN) &&
                            structure.store.getFreeCapacity(RESOURCE_ENERGY) > 0;
                    }
            });
            if(targets.length > 0) {
                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
    }

};

<!-- put energy to tower -->

var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
        else {
            var targets = creep.room.find(FIND_STRUCTURES, {
                    filter: (structure) => {
                        return (structure.structureType == STRUCTURE_EXTENSION ||
                                structure.structureType == STRUCTURE_SPAWN ||
                                structure.structureType == STRUCTURE_TOWER) &&
                                structure.store.getFreeCapacity(RESOURCE_ENERGY) > 0;
                    }
            });
            if(targets.length > 0) {
                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
    }

};

module.exports = roleHarvester;

# role Upgrader

var roleUpgrader = {

    /** @param {Creep} creep **/
    run: function(creep) {
        if(creep.store[RESOURCE_ENERGY] == 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
                creep.moveTo(creep.room.controller);
            }
        }
    }

};

module.exports = roleUpgrader;

# role builder

var roleBuilder = {

    /** @param {Creep} creep **/
    run: function(creep) {

        if(creep.memory.building && creep.store[RESOURCE_ENERGY] == 0) {
            creep.memory.building = false;
            creep.say('🔄 harvest');
        }
        if(!creep.memory.building && creep.store.getFreeCapacity() == 0) {
            creep.memory.building = true;
            creep.say('🚧 build');
        }

        if(creep.memory.building) {
            var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
            if(targets.length) {
                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
        else {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
    }

};

module.exports = roleBuilder;

# console

Game.creeps['Harvester1'].memory.role = 'harvester';
Game.creeps['Upgrader1'].memory.role = 'upgrader';
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Builder1',
{ memory: { role: 'builder' } } );

<!-- 550 energy faster Harvester1 -->

Game.spawns['Spawn1'].spawnCreep( [WORK,WORK,WORK,WORK,CARRY,MOVE,MOVE],
'HarvesterBig',
{ memory: { role: 'harvester' } } )

<!-- sucide creep-->

Game.creeps['Harvester1'].suicide()

<!-- make safe mode. enemy will not attack you -->

Game.spawns['Spawn1'].room.controller.activateSafeMode();

<!-- building tower -->

Game.spawns['Spawn1'].room.createConstructionSite( 23, 22, STRUCTURE_TOWER )

# Script main

var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

<!-- if creep died, you need to clear the memory -->

for(var name in Memory.creeps) {
if(!Game.creeps[name]) {
delete Memory.creeps[name];
console.log('Clearing non-existing creep memory:', name);
}
}

<!-- to count the number of creeps of the required type -->

var harvesters = \_.filter(Game.creeps, (creep) => creep.memory.role == 'harvester');
console.log('Harvesters: ' + harvesters.length);

<!-- set up minimum of creeps working. for now 2 -->

if(harvesters.length < 2) {
var newName = 'Harvester' + Game.time;
console.log('Spawning new harvester: ' + newName);
Game.spawns['Spawn1'].spawnCreep([WORK,CARRY,MOVE], newName,
{memory: {role: 'harvester'}});
}

if(Game.spawns['Spawn1'].spawning) {
var spawningCreep = Game.creeps[Game.spawns['Spawn1'].spawning.name];
Game.spawns['Spawn1'].room.visual.text(
'🛠️' + spawningCreep.memory.role,
Game.spawns['Spawn1'].pos.x + 1,
Game.spawns['Spawn1'].pos.y,
{align: 'left', opacity: 0.8});
}

<!-- to know the total amount of energy in the room -->

for(var name in Game.rooms) {
console.log('Room "'+name+'" has '+Game.rooms[name].energyAvailable+' energy');
}

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }

}

<!-- creating tower's work -->

var tower = Game.getObjectById('28b0c4d974cd84827375842e');
if(tower) {
var closestDamagedStructure = tower.pos.findClosestByRange(FIND_STRUCTURES, {
filter: (structure) => structure.hits < structure.hitsMax
});

<!-- repair structures -->

        if(closestDamagedStructure) {
            tower.repair(closestDamagedStructure);
        }

<!-- command attack for tower -->

        var closestHostile = tower.pos.findClosestByRange(FIND_HOSTILE_CREEPS);
        if(closestHostile) {
            tower.attack(closestHostile);
        }
    }
