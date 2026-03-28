# techspark

class Appliance:
    def __init__(self, name, wattage, daily_hours, standby=False, standby_hours=0):
        self.name = name
        self.wattage = wattage
        self.daily_hours = daily_hours
        self.standby = standby
        self.standby_hours = standby_hours
        def add_appliance(self, appliance):
        self.appliances.append(appliance)
        # create a dictionary of appliances with average wattage
        total_wattage = sum(appliance.wattage for appliance in self.appliances)
        average_wattage = total_wattage / len(self.appliances) if self.appliances else 0
        self.appliance_dict = {appliance.name: appliance.wattage for appliance in self.appliances}
        self.average_wattage = average_wattage
